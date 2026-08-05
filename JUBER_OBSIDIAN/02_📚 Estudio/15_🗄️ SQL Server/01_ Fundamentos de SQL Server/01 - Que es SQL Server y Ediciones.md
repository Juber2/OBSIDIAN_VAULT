---
tags:
  - SQLServer
  - Bases-de-Datos
  - SQLServer-Fundamentos
Creado: 2026-07-29
Relacionado:
  - Fundamentos de SQL Server
  - SQL SERVER
---

# 01 - Que es SQL Server y Ediciones

## Que vas a aprender

En este capitulo vas a entender **que es SQL Server**, el motor relacional de Microsoft, y sus distintas **ediciones**, relevante directamente para tu trabajo en StaffTrainingPlatform y el ecosistema .NET.

## Explicacion

SQL Server es el sistema de gestion de bases de datos relacional de Microsoft, con integracion nativa y profunda en el ecosistema .NET (Entity Framework Core, ya visto de forma introductoria en el libro de NestJS al comparar con TypeORM). A diferencia de PostgreSQL (motor de codigo abierto, ver ese libro complementario), SQL Server tiene licenciamiento comercial con varias ediciones: **Express** (gratuita, con limites de tamaño de base de datos y recursos, apropiada para desarrollo y aplicaciones chicas), **Standard** (licenciada, para produccion de tamaño medio), **Enterprise** (licenciada, con todas las features avanzadas: Always On, particionamiento completo, In-Memory OLTP sin restricciones), y **Developer** (gratuita, con todas las features de Enterprise, pero solo licenciada para desarrollo, nunca produccion).

Para un proyecto como StaffTrainingPlatform en desarrollo, la edicion **Developer** es la eleccion natural: da acceso completo a features enterprise sin costo, ideal para aprender y probar capacidades avanzadas antes de decidir que edicion de produccion se necesita realmente.

## Modelo mental

Pensa las ediciones de SQL Server como distintos niveles de membresia en un club: Express es la membresia gratuita basica con acceso limitado a las instalaciones, Developer es una membresia completa gratuita pero solo para "practicar" (no para uso comercial real), y Standard/Enterprise son las membresias pagas con acceso completo (Enterprise) o parcial (Standard) a las instalaciones premium, para uso real del negocio.

## Ejemplo base

```sql
SELECT @@VERSION;   -- version y edicion exacta instalada

SELECT SERVERPROPERTY('Edition');    -- solo el nombre de la edicion
SELECT SERVERPROPERTY('ProductVersion');
```

## Como se usa en un proyecto real

1. Se usa **Developer Edition** para desarrollo local de proyectos .NET como StaffTrainingPlatform, aprovechando features completas sin costo.
2. Se evalua **Standard** versus **Enterprise** para produccion segun features realmente necesarias (Always On basico esta en Standard, particionamiento completo requiere Enterprise en versiones mas antiguas, aunque esto cambio en versiones recientes).
3. Se verifica la edicion real de un servidor de produccion antes de asumir disponibilidad de una feature especifica (`SERVERPROPERTY('Edition')`).
4. Se documenta que edicion usa cada ambiente (desarrollo, staging, produccion) del proyecto.
5. Se conecta esto con Azure SQL Database (ver capitulo de Nube), una alternativa gestionada que abstrae completamente el concepto de edicion tradicional con su propio modelo de tiers.

## Criterio junior

Un junior debe poder identificar la edicion y version instalada con `SELECT @@VERSION`.

## Criterio mid

Un mid debe elegir la edicion apropiada para desarrollo (Developer) versus produccion (Standard/Enterprise) segun necesidades reales.

## Criterio senior

Un senior debe decidir la estrategia de licenciamiento de SQL Server para toda la organizacion, evaluando el costo real de features enterprise necesarias frente a alternativas como Azure SQL Database gestionado.

## Errores comunes

- Usar Express Edition para un proyecto que crecera mas alla de sus limites de tamaño, sin planificar la migracion a una edicion superior.
- Usar Developer Edition en produccion, violando los terminos de licencia (solo permitida para desarrollo).
- No verificar la edicion real de un servidor antes de asumir disponibilidad de una feature enterprise especifica.
- No considerar Azure SQL Database como alternativa que simplifica la decision de edicion con su propio modelo de licenciamiento por consumo.

## Practica

- [ ] Instala SQL Server Developer Edition (o usa el container de Docker, ver capitulo correspondiente) y verifica la version con `@@VERSION`.
- [ ] Investiga las diferencias de features entre Standard y Enterprise relevantes para tu contexto.
- [ ] Verifica que edicion usa (o usaria) StaffTrainingPlatform en cada ambiente.
- [ ] Compara el modelo de licenciamiento tradicional con Azure SQL Database.
- [ ] Documenta que edicion recomendarias para produccion de un proyecto real tuyo.

## Preguntas de repaso

- Que diferencia hay entre Developer Edition y Express Edition de SQL Server?
- Por que Developer Edition nunca deberia usarse en produccion pese a tener features completas?
- Que verificarias antes de asumir que un servidor de produccion soporta una feature especifica?
- Como simplifica Azure SQL Database la decision tradicional de edicion?
