---
tags:
  - SQLServer
  - Nube
  - SQLServer-Nube
Creado: 2026-07-29
Relacionado:
  - SQL Server en la Nube y Contenedores
  - SQL SERVER
---

# 04 - Migraciones EF Core en la Nube

## Que vas a aprender

En este capitulo vas a aprender como aplicar migraciones de Entity Framework Core de forma segura contra una base de datos SQL Server en la nube (Azure SQL Database o SQL Server en contenedores/Kubernetes), un proceso con consideraciones distintas al desarrollo local.

## Explicacion

Aplicar `dotnet ef database update` directamente contra una base de datos de produccion en la nube desde la maquina de un desarrollador es una practica riesgosa: expone credenciales de produccion en el entorno local, no deja rastro auditable de quien aplico que migracion y cuando, y no se integra con revision de codigo. El patron recomendado es generar un **script SQL idempotente** con `dotnet ef migrations script --idempotent`, revisarlo como parte de un Pull Request, y aplicarlo desde un pipeline de CI/CD con credenciales gestionadas de forma segura (nunca hardcodeadas), el mismo principio de migraciones auditable y controladas ya aplicable identicamente a Postgres con TypeORM/Prisma (ver ese libro).

Para Azure SQL Database especificamente, es importante considerar el **tamaño de la transaccion** de una migracion sobre una tabla grande: alterar una tabla con millones de filas puede consumir mucho **log space** dentro de los limites del tier contratado, y una migracion mal diseñada puede fallar a mitad de camino o consumir mas DTU/vCore de los previstos, afectando produccion mientras corre.

## Modelo mental

Pensa aplicar una migracion EF Core contra la nube como firmar un documento legal importante: no lo haces informalmente desde tu escritorio sin testigos (tu maquina local con credenciales de produccion), lo llevas a traves de un proceso formal con revision y registro (Pull Request + pipeline de CI/CD), dejando un rastro claro de quien aplico que cambio y cuando.

## Ejemplo base

```bash
# Generar script SQL idempotente en vez de aplicar directo
dotnet ef migrations script --idempotent -o migracion.sql

# El script generado revisa el estado actual antes de aplicar cada cambio,
# seguro de ejecutar multiples veces sin duplicar cambios ya aplicados
```

```yaml
# Paso simplificado en un pipeline de CI/CD (Azure DevOps/GitHub Actions)
- name: Aplicar migraciones EF Core
  run: |
    dotnet ef migrations script --idempotent -o migracion.sql
    sqlcmd -S $env:SQL_SERVER -d $env:SQL_DATABASE -U $env:SQL_USER -P $env:SQL_PASSWORD -i migracion.sql
  env:
    SQL_SERVER: ${{ secrets.SQL_SERVER }}
    SQL_PASSWORD: ${{ secrets.SQL_PASSWORD }}
```

## Como se usa en un proyecto real

1. Se genera siempre un script SQL idempotente para migraciones de StaffTrainingPlatform antes de aplicarlas contra Azure SQL Database, en vez de ejecutar `database update` directo desde la maquina local.
2. Se revisa el script generado como parte del Pull Request, permitiendo que otro desarrollador detecte cambios destructivos (como un `DROP COLUMN` no intencional) antes de aplicarlos.
3. Se aplica el script desde un pipeline de CI/CD con credenciales gestionadas como secretos, nunca hardcodeadas en el codigo o en configuracion versionada.
4. Se evalua el impacto de una migracion sobre tablas grandes en terminos de consumo de recursos del tier de Azure SQL Database contratado, evitando migraciones que degraden produccion mientras corren.
5. Se conecta esto con el principio de migraciones zero-downtime ya visto en el capitulo Senior de Postgres, aplicando el mismo cuidado a cambios de esquema en SQL Server.

## Criterio junior

Un junior debe poder generar un script de migracion idempotente con `dotnet ef migrations script`.

## Criterio mid

Un mid debe integrar la aplicacion de migraciones dentro de un pipeline de CI/CD con credenciales seguras.

## Criterio senior

Un senior debe evaluar el impacto de recursos de una migracion sobre tablas grandes en un tier de Azure SQL Database especifico, y diseñar migraciones zero-downtime cuando sea necesario.

## Errores comunes

- Ejecutar `dotnet ef database update` directamente contra produccion desde una maquina local, sin rastro auditable ni revision.
- Hardcodear credenciales de la base de datos de produccion en el codigo o en archivos de configuracion versionados.
- No revisar el script SQL generado antes de aplicarlo, dejando pasar cambios destructivos no intencionales.
- No considerar el consumo de recursos de una migracion sobre una tabla grande dentro de los limites del tier contratado.

## Practica

- [ ] Genera un script SQL idempotente con `dotnet ef migrations script --idempotent`.
- [ ] Revisa el script generado identificando que cambios especificos aplicaria.
- [ ] Diseña un paso de pipeline de CI/CD que aplique este script con credenciales seguras.
- [ ] Investiga el impacto de una migracion sobre una tabla grande en un tier especifico de Azure SQL Database.
- [ ] Compara este proceso con migraciones zero-downtime de Postgres ya estudiadas.

## Preguntas de repaso

- Por que es riesgoso aplicar migraciones directamente desde una maquina local contra produccion?
- Que ventaja tiene un script idempotente sobre `database update` directo?
- Que riesgo especifico introduce una migracion sobre una tabla grande en Azure SQL Database?
- Como se conecta este proceso con el principio de migraciones zero-downtime ya visto en Postgres?
