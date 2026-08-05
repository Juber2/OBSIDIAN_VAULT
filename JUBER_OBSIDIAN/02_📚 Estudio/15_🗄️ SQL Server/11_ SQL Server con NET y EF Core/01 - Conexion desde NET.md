---
tags:
  - SQLServer
  - DotNet
  - SQLServer-DotNet
Creado: 2026-07-29
Relacionado:
  - SQL Server con NET y EF Core
  - SQL SERVER
---

# 01 - Conexion desde NET

## Que vas a aprender

En este capitulo vas a aprender como .NET se conecta a SQL Server: `Microsoft.Data.SqlClient` como driver base, cadenas de conexion, y **connection pooling** automatico, sentando la base antes de entrar a Entity Framework Core.

## Explicacion

`Microsoft.Data.SqlClient` es el driver ADO.NET moderno para SQL Server (sucesor de `System.Data.SqlClient`), la capa mas baja sobre la que Entity Framework Core se construye, de forma analoga a como Npgsql es el driver base para Postgres en .NET. Una cadena de conexion tipica especifica servidor, base de datos, credenciales (o `Integrated Security=true` para autenticacion de Windows), y parametros de seguridad como `Encrypt=True` (obligatorio para conexiones hacia Azure SQL Database, ver capitulo anterior).

.NET gestiona un **connection pool** automaticamente: abrir y cerrar conexiones frecuentemente no crea/destruye conexiones TCP reales cada vez, sino que las reutiliza desde un pool mantenido internamente por el driver, un mecanismo transparente para el desarrollador pero importante entender porque afecta directamente el dimensionamiento del `Max Pool Size` en sistemas de alta concurrencia, y porque no cerrar conexiones explicitamente (`using`) puede agotar el pool disponible.

## Modelo mental

Pensa el connection pool como una fila de taxis esperando en una parada: en vez de fabricar un auto nuevo cada vez que alguien necesita viajar (abrir una conexion TCP nueva, costoso), un taxi ya existente y libre te lleva y vuelve a la fila para el proximo pasajero (la conexion se devuelve al pool al cerrarse, lista para reutilizarse). Si todos los taxis estan ocupados y nadie los devuelve (conexiones no cerradas correctamente), los nuevos pasajeros quedan esperando indefinidamente.

## Ejemplo base

```csharp
using Microsoft.Data.SqlClient;

var connectionString = "Server=localhost,1433;Database=MiApp;" +
    "User Id=sa;Password=ContraseñaSegura123!;Encrypt=True;TrustServerCertificate=True;";

using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();

using var command = new SqlCommand("SELECT Nombre FROM Cursos WHERE Id = @Id", connection);
command.Parameters.AddWithValue("@Id", 1);

using var reader = await command.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    Console.WriteLine(reader.GetString(0));
}
// La conexion vuelve al pool automaticamente al salir del using
```

## Como se usa en un proyecto real

1. Se define la cadena de conexion de StaffTrainingPlatform en `appsettings.json` (o variables de entorno/secretos para produccion), nunca hardcodeada en el codigo fuente.
2. Se usa siempre `using` (o `await using`) para garantizar que las conexiones vuelvan al pool inmediatamente despues de usarse, evitando agotamiento del pool bajo carga.
3. Se dimensiona `Max Pool Size` en la cadena de conexion segun la concurrencia esperada de la aplicacion, evitando timeouts por pool agotado en momentos de alto trafico.
4. Se usa siempre parametros (`@Id` en el ejemplo) en vez de concatenar valores directamente en el SQL, la misma prevencion de SQL injection ya insistida en el tema de Dynamic SQL.
5. Se entiende que Entity Framework Core (siguiente tema) construye toda su funcionalidad sobre esta misma capa base de `Microsoft.Data.SqlClient`.

## Criterio junior

Un junior debe poder abrir una conexion, ejecutar un comando parametrizado, y leer resultados con `SqlDataReader`.

## Criterio mid

Un mid debe entender el connection pooling y garantizar que las conexiones se cierren correctamente en todo el codigo.

## Criterio senior

Un senior debe dimensionar el pool de conexiones apropiadamente para la concurrencia real del sistema y diagnosticar problemas de agotamiento de pool en produccion.

## Errores comunes

- No usar `using`/`await using`, dejando conexiones abiertas que agotan el pool bajo carga.
- Concatenar valores directamente en el SQL en vez de usar parametros, exponiendo la aplicacion a SQL injection.
- Hardcodear la cadena de conexion (incluyendo credenciales) directamente en el codigo fuente.
- No dimensionar `Max Pool Size` para la concurrencia real, causando timeouts intermitentes dificiles de diagnosticar bajo carga alta.

## Practica

- [ ] Conecta una aplicacion .NET a SQL Server usando `Microsoft.Data.SqlClient` directamente (sin EF Core).
- [ ] Ejecuta una query parametrizada y lee los resultados con `SqlDataReader`.
- [ ] Investiga como configurar `Max Pool Size` en una cadena de conexion.
- [ ] Provoca intencionalmente un agotamiento de pool no cerrando conexiones, y observa el comportamiento.
- [ ] Compara este driver con Npgsql para Postgres en .NET.

## Preguntas de repaso

- Que problema resuelve el connection pooling, y por que es importante para performance?
- Que riesgo introduce no cerrar conexiones explicitamente con `using`?
- Por que se deben usar parametros en vez de concatenar valores directamente en el SQL?
- Como se relaciona `Microsoft.Data.SqlClient` con Entity Framework Core?
