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

# 02 - Azure SQL Database

## Que vas a aprender

En este capitulo vas a aprender **Azure SQL Database**, el servicio administrado (PaaS) de SQL Server en la nube de Microsoft, equivalente directo a Amazon RDS/Aurora para Postgres ya estudiado en ese libro.

## Explicacion

Azure SQL Database es una **base de datos como servicio**: Microsoft administra el parcheo del motor, backups automaticos con PITR incorporado, y alta disponibilidad basica, dejando al equipo enfocarse solo en el modelado de datos y las queries, exactamente el mismo valor que ofrecen los servicios administrados de Postgres ya estudiados. Existen dos modelos de despliegue principales: **Single Database** (una base de datos aislada con sus propios recursos, el modelo mas comun) y **Elastic Pool** (varias bases de datos comparten un pool de recursos, eficiente cuando hay muchas bases de datos con patrones de uso variables, como en aplicaciones multi-tenant).

El nivel de servicio se elige mediante **tiers de compra**: **DTU-based** (un modelo simplificado que combina CPU, memoria, e I/O en una sola metrica) o **vCore-based** (mas granular y comparable directamente con el hardware subyacente, la opcion recomendada para cargas de produccion serias porque permite razonar sobre CPU y memoria por separado). Azure SQL Database tambien ofrece **Hyperscale**, un tier diseñado para bases de datos muy grandes con escalado de almacenamiento casi ilimitado y separacion de computo/almacenamiento.

## Modelo mental

Pensa Azure SQL Database exactamente igual que RDS/Aurora para Postgres ya estudiado: "alguien mas se encarga de parchear, respaldar, y mantener el motor corriendo, vos solo te preocupas por tus tablas y queries." La eleccion entre DTU-based y vCore-based es como elegir entre un plan de datos con "categoria general" (DTU, mas simple) o uno donde eliges exactamente cuantos nucleos y cuanta memoria queres (vCore, mas control).

## Ejemplo base

```bash
# Crear un Azure SQL Database via Azure CLI
az sql server create \
    --name miapp-sqlserver \
    --resource-group MiGrupoRecursos \
    --admin-user adminuser \
    --admin-password 'ContraseñaSegura123!'

az sql db create \
    --resource-group MiGrupoRecursos \
    --server miapp-sqlserver \
    --name MiApp \
    --service-objective S1
```

```
Cadena de conexion tipica para .NET/EF Core:
Server=tcp:miapp-sqlserver.database.windows.net,1433;
Database=MiApp;
User ID=adminuser;
Password=ContraseñaSegura123!;
Encrypt=True;
```

## Como se usa en un proyecto real

1. Se elige Azure SQL Database para produccion de StaffTrainingPlatform cuando se prioriza no gestionar infraestructura de backups/parcheo manualmente, delegandolo al servicio administrado.
2. Se elige el modelo vCore-based para produccion (control granular de CPU/memoria) en vez de DTU-based, que es mas apropiado solo para cargas pequeñas o de prueba.
3. Se usa Elastic Pool si el sistema es multi-tenant con muchas bases de datos de uso variable, optimizando costos frente a aprovisionar cada una individualmente.
4. Se conecta la cadena de conexion con Entity Framework Core (ver capitulo de SQL Server con .NET), asegurando el uso de `Encrypt=True` para conexiones seguras hacia Azure.
5. Se compara el costo total de propiedad frente a administrar SQL Server manualmente en una VM o contenedor, considerando el tiempo de ingenieria ahorrado en mantenimiento.

## Criterio junior

Un junior debe poder crear un Azure SQL Database basico y conectarse desde una aplicacion .NET.

## Criterio mid

Un mid debe elegir el tier de servicio apropiado (DTU vs vCore) segun los requisitos de carga del sistema.

## Criterio senior

Un senior debe diseñar la arquitectura de costos y escalado (Single Database, Elastic Pool, o Hyperscale) segun el patron de uso real del sistema.

## Errores comunes

- Usar DTU-based en produccion cuando vCore-based ofreceria mejor control y previsibilidad de performance.
- No usar Elastic Pool en sistemas multi-tenant con muchas bases de datos, pagando de mas por recursos aprovisionados individualmente.
- No configurar `Encrypt=True` en la cadena de conexion, exponiendo trafico sin cifrar hacia el servicio en la nube.
- Subestimar el costo de Azure SQL Database frente a gestionar SQL Server manualmente, sin considerar el ahorro real en tiempo de ingenieria.

## Practica

- [ ] Crea un Azure SQL Database de prueba (o investiga el proceso en el portal de Azure).
- [ ] Compara los tiers DTU-based y vCore-based en terminos de que metricas expone cada uno.
- [ ] Investiga cuando conviene usar Elastic Pool frente a Single Database.
- [ ] Configura una cadena de conexion segura desde un proyecto .NET hacia Azure SQL Database.
- [ ] Compara Azure SQL Database con RDS/Aurora de Postgres, identificando las equivalencias.

## Preguntas de repaso

- Que diferencia hay entre el modelo DTU-based y vCore-based?
- Cuando conviene usar Elastic Pool en vez de Single Database?
- Que es Hyperscale y para que tipo de cargas esta diseñado?
- Como se compara Azure SQL Database con RDS/Aurora de Postgres?
