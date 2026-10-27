---
tags:
  - Azure
  - Seguridad
  - KeyVault
Creado: 2026-08-30
Relacionado:
  - Azure
  - Laboratorios
---

# Key Vault y Managed Identity

> Pregunta literal de la vacante de Entelgy: **"¿Cómo accede App Service a Key Vault sin secretos?"**. Esta nota existe para que puedas responderla con la cadena completa, no con "usa Key Vault".

## El problema que resuelve

```json
// ❌ appsettings.json commiteado al repo
{
  "ConnectionStrings": {
    "Default": "Server=tcp:midb.database.windows.net;User ID=admin;Password=P@ssw0rd123"
  }
}
```

Mover ese secreto a una variable de entorno del App Service es **mejor pero no suficiente**: sigue visible en el portal, en el pipeline, y para cualquiera con permisos de lectura de configuración. Key Vault lo saca de ahí; Managed Identity elimina la necesidad de un secreto *para acceder a Key Vault* (si no, solo moviste el problema un nivel).

## La cadena completa (esto es lo que hay que poder explicar)

```
App Service
   │  tiene una Managed Identity (Azure le asigna una identidad en Entra ID)
   ▼
Entra ID  ──► emite un token para esa identidad, sin password
   │
   ▼
Key Vault ──► valida el token + revisa RBAC ──► devuelve el secreto
```

**El punto clave:** en ningún paso hay una credencial almacenada por la aplicación. Azure inyecta la identidad en la plataforma (el runtime la obtiene de un endpoint local, `IMDS`), no en el código ni en la configuración.

## Paso 1 — Crear el Key Vault

```bash
az keyvault create \
  --name mi-kv-prod \
  --resource-group mi-rg \
  --location eastus \
  --enable-rbac-authorization true
```

`--enable-rbac-authorization true` usa el **modelo RBAC** (recomendado hoy) en vez de las Access Policies legacy. Diferencia práctica: RBAC se gestiona con los mismos roles de Azure que el resto de recursos; las Access Policies son un sistema de permisos aparte, solo del Key Vault.

```bash
az keyvault secret set --vault-name mi-kv-prod --name "DbConnectionString" --value "Server=tcp:..."
```

## Paso 2 — Darle identidad al App Service

```bash
# System-assigned: nace y muere con el recurso
az webapp identity assign --resource-group mi-rg --name mi-api

# Devuelve un principalId — guardalo para el paso 3
```

### System-assigned vs User-assigned

| | System-assigned | User-assigned |
|---|---|---|
| Ciclo de vida | Atada al recurso; se borra con él | Recurso independiente, sobrevive |
| Cuántos recursos la usan | Uno solo | Varios comparten la misma |
| Cuándo usarla | Caso simple, un servicio | Varias apps que necesitan los mismos permisos, o cuando querés asignar permisos *antes* de crear la app |

En entrevista: si te preguntan cuál elegir, la respuesta correcta es "depende de si más de un recurso necesita la misma identidad" — no hay una universalmente mejor.

## Paso 3 — Dar permiso a esa identidad sobre el Key Vault

```bash
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <principalId-del-paso-2> \
  --scope /subscriptions/<sub-id>/resourceGroups/mi-rg/providers/Microsoft.KeyVault/vaults/mi-kv-prod
```

`Key Vault Secrets User` solo permite **leer** secretos. Es el rol correcto para una app en runtime — mínimo privilegio. `Key Vault Administrator` es para vos como operador, nunca para la aplicación.

## Paso 4a — Consumirlo desde la aplicación (.NET)

```csharp
// Program.cs
var keyVaultUri = new Uri($"https://mi-kv-prod.vault.azure.net/");
builder.Configuration.AddAzureKeyVault(keyVaultUri, new DefaultAzureCredential());

// A partir de acá el secreto se lee como configuración normal
var connectionString = builder.Configuration["DbConnectionString"];
```

`DefaultAzureCredential` es la pieza clave: en Azure usa la Managed Identity automáticamente; en tu máquina usa tus credenciales de `az login` o Visual Studio. **El mismo código funciona local y en producción sin cambios ni `if (isProduction)`.**

## Paso 4b — Consumirlo desde la aplicación (Java/Spring)

```java
SecretClient client = new SecretClientBuilder()
    .vaultUrl("https://mi-kv-prod.vault.azure.net/")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();

String connectionString = client.getSecret("DbConnectionString").getValue();
```

Con Spring Cloud Azure el equivalente declarativo:

```properties
spring.cloud.azure.keyvault.secret.property-sources[0].endpoint=https://mi-kv-prod.vault.azure.net/
spring.cloud.azure.credential.managed-identity-enabled=true
```

Y el secreto queda disponible con `@Value("${DbConnectionString}")` como cualquier property.

## Atajo: referencias de Key Vault en App Settings

Si no querés tocar el código en absoluto:

```bash
az webapp config appsettings set --resource-group mi-rg --name mi-api \
  --settings DbConnectionString="@Microsoft.KeyVault(SecretUri=https://mi-kv-prod.vault.azure.net/secrets/DbConnectionString/)"
```

App Service resuelve la referencia al arrancar y la app ve una variable de entorno normal. Útil para migrar una app existente sin refactor — pero **se resuelve solo al iniciar**, así que rotar el secreto exige reiniciar la app. Con el SDK podés recargar sin reinicio.

## Rotación

Un secreto en Key Vault tiene versiones. Si la referencia apunta al secreto **sin versión** (como arriba), la app toma la última al reiniciar. Si apunta a una versión específica, queda fijada — útil para control estricto, peligroso si olvidás actualizarla.

Para bases de datos, la alternativa superior es no tener password: **autenticación de Azure SQL con Managed Identity**, donde ni siquiera existe un secreto que rotar (ver [[05 - Azure SQL y Blob Storage]]).

## Errores comunes

- Guardar el secreto en Key Vault pero autenticarse **contra Key Vault** con un client secret — el secreto solo cambió de lugar. Managed Identity es lo que rompe la cadena.
- Usar `Key Vault Administrator` para la app "porque así funciona seguro" — viola mínimo privilegio y es un hallazgo directo en cualquier auditoría.
- Asumir que la referencia `@Microsoft.KeyVault(...)` se actualiza sola al rotar. No: requiere reinicio.
- Probar en local con un client secret hardcodeado y en producción con Managed Identity: dos rutas de código distintas. `DefaultAzureCredential` existe justamente para evitarlo.
- Olvidar que Managed Identity **no funciona fuera de Azure** — en un pipeline de GitHub Actions se usa OIDC/federated credentials, no Managed Identity (ver [[GitHub Actions CI CD]]).

## Checklist

- [ ] Puedo explicar la cadena App Service → Entra ID → Key Vault sin saltarme pasos.
- [ ] Sé la diferencia entre system-assigned y user-assigned, y cuándo elegir cada una.
- [ ] Sé qué rol RBAC mínimo necesita una app para leer secretos.
- [ ] Entiendo por qué `DefaultAzureCredential` permite el mismo código en local y en Azure.
- [ ] Sé el trade-off entre referencia de App Settings y SDK (reinicio vs recarga).
- [ ] Puedo explicar por qué autenticarse a Azure SQL con Managed Identity es mejor que guardar el password en Key Vault.
