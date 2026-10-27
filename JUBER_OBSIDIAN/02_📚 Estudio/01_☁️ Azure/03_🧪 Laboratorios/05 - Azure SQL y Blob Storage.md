---
tags:
  - Azure
  - AzureSQL
  - Storage
Creado: 2026-08-30
Relacionado:
  - Azure
  - Laboratorios
---

# Azure SQL y Blob Storage

Bases relacionales aparecen en 11 de 14 vacantes del radar y object storage en 2. Ya tenés 85 notas de SQL Server y 85 de PostgreSQL: esta nota cubre **solo lo que cambia al pasar a la nube**, no SQL desde cero.

## Azure SQL: qué cambia respecto de SQL Server on-premise

| | SQL Server on-prem | Azure SQL Database |
|---|---|---|
| Backups | Los configurás vos | Automáticos, con point-in-time restore |
| Alta disponibilidad | Always On, cluster manual | Incluida según tier |
| Parches y versión | Los gestionás vos | Azure los aplica |
| `USE otra_base` | Funciona | **No** — cada base es un recurso aislado |
| SQL Agent Jobs | Disponible | **No existe** — se reemplaza con Elastic Jobs, Functions o Logic Apps |
| Cross-database queries | Directas | Requieren External Tables |
| Costo | CAPEX del servidor | Por tier, por hora |

Las dos filas que sorprenden en un proyecto real —y por eso preguntan— son **SQL Agent** y **cross-database**: código que funcionaba on-prem deja de funcionar tal cual.

> Si el proyecto necesita SQL Agent, instancia completa o compatibilidad total, la opción es **Azure SQL Managed Instance**, no Azure SQL Database. Saber que existen los tres modelos (VM, Managed Instance, Database) y cuándo elegir cada uno es exactamente el tipo de criterio que evalúan.

## Modelos de compra: DTU vs vCore

| Modelo | Qué es | Cuándo |
|---|---|---|
| **DTU** | Unidad mezclada de CPU + memoria + I/O | Cargas simples, presupuesto predecible |
| **vCore** | Elegís cores y memoria por separado | Producción; permite Azure Hybrid Benefit (reusar licencias) |
| **Serverless** | Escala solo y **se pausa** sin uso | Dev/test o cargas intermitentes — pagás cómputo solo cuando corre |

```bash
az sql server create --name mi-sql-srv --resource-group mi-rg \
  --location eastus --enable-ad-only-auth --external-admin-name "grupo-dba"

az sql db create --resource-group mi-rg --server mi-sql-srv --name ordenes \
  --edition GeneralPurpose --family Gen5 --capacity 2
```

> Serverless tiene **cold start**: la primera consulta después de la pausa tarda. Barato para dev, inaceptable para una API de cara al usuario.

## Conexión sin password (lo que quieren escuchar)

Guardar el password en Key Vault es bueno. **No tener password es mejor:**

```bash
az sql server ad-admin create --resource-group mi-rg --server mi-sql-srv \
  --display-name "grupo-dba" --object-id <object-id>
```

```sql
-- Dentro de la base: dar acceso a la Managed Identity de la app
CREATE USER [mi-api] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [mi-api];
ALTER ROLE db_datawriter ADD MEMBER [mi-api];
```

```csharp
// Sin usuario ni password en la cadena de conexión
"Server=tcp:mi-sql-srv.database.windows.net,1433;Database=ordenes;Authentication=Active Directory Default;"
```

```properties
# Java / Spring
spring.datasource.url=jdbc:sqlserver://mi-sql-srv.database.windows.net:1433;database=ordenes;authentication=ActiveDirectoryMSI
```

Cadena completa: la app usa su Managed Identity → Entra ID emite el token → Azure SQL lo valida. **No hay secreto que rotar, filtrar ni guardar** — es el argumento que cierra la pregunta "¿cómo manejás credenciales de base de datos?".

## Resiliencia de conexión: obligatoria en la nube

En la nube, la base **se mueve**: failover, mantenimiento, throttling. Los errores transitorios no son excepcionales, son parte del funcionamiento normal.

```csharp
options.UseSqlServer(connectionString, sql =>
{
    sql.EnableRetryOnFailure(
        maxRetryCount: 5,
        maxRetryDelay: TimeSpan.FromSeconds(10),
        errorNumbersToAdd: null);
});
```

```java
// HikariCP + reintentos ante fallos transitorios
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.maximum-pool-size=10
```

> **Trampa conocida:** `EnableRetryOnFailure` es incompatible con transacciones iniciadas manualmente (`BeginTransaction`) salvo que uses una estrategia de ejecución explícita. Es un error muy común al migrar de on-prem, donde nadie configuraba reintentos.

Los códigos de error transitorios típicos de Azure SQL: **40197, 40501, 40613, 49918, 4060**. No hace falta memorizarlos, pero sí saber que existe una lista y que la resiliencia se basa en ella.

## Red: firewall y private endpoint

```bash
# Permitir servicios de Azure (suficiente para empezar)
az sql server firewall-rule create --resource-group mi-rg --server mi-sql-srv \
  --name AllowAzureServices --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

Esa regla especial `0.0.0.0-0.0.0.0` significa "cualquier servicio de Azure" — **incluye suscripciones de otros clientes**. Para producción, lo correcto es **Private Endpoint**: la base recibe una IP privada dentro de tu red virtual y deja de ser accesible desde internet.

## Blob Storage

```bash
az storage account create --name miappstorage --resource-group mi-rg \
  --sku Standard_LRS --kind StorageV2 --min-tls-version TLS1_2

az storage container create --account-name miappstorage --name facturas --auth-mode login
```

### Tiers de acceso

| Tier | Costo de almacenar | Costo de leer | Uso |
|---|---|---|---|
| Hot | Alto | Bajo | Acceso frecuente |
| Cool | Medio | Medio | Acceso mensual (mín. 30 días) |
| Cold | Bajo | Alto | Acceso trimestral (mín. 90 días) |
| Archive | Muy bajo | Muy alto + **horas de rehidratación** | Retención legal |

Archive no se lee: se *rehidrata*, y tarda horas. Poner ahí algo que la aplicación necesita en línea es un error de diseño costoso.

### Subir y leer

```csharp
var client = new BlobServiceClient(
    new Uri("https://miappstorage.blob.core.windows.net"),
    new DefaultAzureCredential());

var container = client.GetBlobContainerClient("facturas");
await container.GetBlobClient("factura-123.pdf").UploadAsync(stream, overwrite: true);
```

```java
BlobServiceClient client = new BlobServiceClientBuilder()
    .endpoint("https://miappstorage.blob.core.windows.net")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();

client.getBlobContainerClient("facturas")
      .getBlobClient("factura-123.pdf")
      .upload(stream, true);
```

### SAS: dar acceso temporal sin exponer la cuenta

Cuando un usuario debe descargar un archivo directamente, la peor solución es que la API descargue el blob y lo reenvíe (consume ancho de banda y memoria del servidor). La correcta es un **SAS token**: una URL firmada, temporal y de permiso limitado.

```csharp
var sasBuilder = new BlobSasBuilder
{
    BlobContainerName = "facturas",
    BlobName = "factura-123.pdf",
    Resource = "b",
    ExpiresOn = DateTimeOffset.UtcNow.AddMinutes(15)
};
sasBuilder.SetPermissions(BlobSasPermissions.Read);   // solo lectura, 15 minutos
```

Reglas: **expiración corta**, **permiso mínimo** (read, no write), y preferir **User Delegation SAS** (firmado con Entra ID) sobre el firmado con la clave de la cuenta — si se filtra la clave de cuenta, se compromete todo el storage.

## Errores comunes

- Migrar de on-prem asumiendo que SQL Agent y cross-database queries van a funcionar.
- No activar reintentos ante fallos transitorios, y luego culpar a "la nube inestable".
- Combinar `EnableRetryOnFailure` con transacciones manuales sin estrategia de ejecución.
- Dejar `AllowAzureServices` en producción creyendo que restringe a tu suscripción.
- Guardar el password de la base cuando Managed Identity elimina la necesidad de tenerlo.
- Servir archivos a través de la API en lugar de emitir un SAS.
- Emitir SAS con expiración larga o permisos de escritura innecesarios.
- Poner en Archive datos que la aplicación consulta en línea.

## Checklist

- [ ] Sé qué deja de funcionar al migrar de SQL Server on-prem a Azure SQL Database.
- [ ] Sé cuándo elegir SQL Database, Managed Instance o SQL en VM.
- [ ] Puedo explicar la conexión a Azure SQL con Managed Identity, sin password.
- [ ] Sé por qué los reintentos son obligatorios en la nube y cuál es la trampa con transacciones.
- [ ] Sé la diferencia entre firewall rule y private endpoint, y cuál va a producción.
- [ ] Puedo explicar los tiers de blob y por qué Archive no sirve para acceso en línea.
- [ ] Sé qué es un SAS, sus tres reglas de uso seguro, y por qué es mejor que proxear el archivo.
