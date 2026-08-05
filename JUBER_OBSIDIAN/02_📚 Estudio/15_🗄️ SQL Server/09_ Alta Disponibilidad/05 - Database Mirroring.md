---
tags:
  - SQLServer
  - AltaDisponibilidad
  - SQLServer-AltaDisponibilidad
Creado: 2026-07-29
Relacionado:
  - Alta Disponibilidad
  - SQL SERVER
---

# 05 - Database Mirroring

## Que vas a cerrar

En este capitulo vas a cerrar el capitulo aprendiendo **Database Mirroring**, la tecnologia predecesora de Always On AG (hoy en estado **deprecated**, oficialmente reemplazada), estudiada para entender su historia y reconocerla en sistemas legacy que aun puedas encontrar en produccion.

## Explicacion

Database Mirroring funcionaba de forma similar a Always On AG pero limitado a **una sola base de datos por sesion de mirroring** (sin agrupar multiples bases de datos como un Availability Group) y **sin readable secondaries reales** (solo permitia un "database snapshot" de solo lectura sobre el espejo, no lecturas continuas). Microsoft lo marco como deprecated desde SQL Server 2012, coincidiendo con la introduccion de Always On AG como su reemplazo oficial, mas flexible y con mas capacidades (multiples bases de datos agrupadas, replicas legibles reales, mas de un secundario).

Es importante reconocer Database Mirroring principalmente porque **sigue existiendo en sistemas legacy** que no fueron migrados, y un profesional senior debe poder identificarlo, entender sus limitaciones frente a las alternativas modernas, y planificar su migracion hacia Always On AG cuando se encuentra en un ambiente de produccion antiguo.

## Modelo mental

Pensa Database Mirroring como el modelo de telefono con cable, funcional para su epoca pero limitado (una sola linea, sin funciones extra), reemplazado hace mas de una decada por un modelo mas capaz (Always On AG, equivalente al smartphone: multiples lineas agrupadas, capacidades adicionales como lectura en el secundario). Si encontras uno en produccion hoy, es una señal clara de que el sistema necesita modernizacion.

## Ejemplo base

```sql
-- Sintaxis historica de Database Mirroring (deprecated, solo para reconocimiento)
ALTER DATABASE MiApp
SET PARTNER = 'TCP://sqlnode2:5023';

-- Verificar si una base de datos usa mirroring (en sistemas legacy)
SELECT name, mirroring_state_desc
FROM sys.database_mirroring
WHERE mirroring_state IS NOT NULL;
```

## Como se usa en un proyecto real

1. Se identifica Database Mirroring en sistemas legacy durante una auditoria de infraestructura, reconociendolo como tecnologia deprecated que requiere plan de migracion.
2. Se planifica la migracion de Database Mirroring hacia Always On AG, documentando el proceso dado que no es una migracion automatica.
3. Se explica a stakeholders no tecnicos por que modernizar esta pieza de infraestructura es necesario, aunque "funcione" actualmente.
4. Se conecta este conocimiento historico con la evolucion general de las tecnologias de alta disponibilidad de SQL Server, para entender el contexto de por que Always On AG existe con su diseño actual.
5. Se evita implementar Database Mirroring en sistemas nuevos, usando siempre Always On AG como la opcion moderna correspondiente.

## Criterio junior

Un junior debe poder reconocer Database Mirroring en un sistema legacy y saber que esta deprecated.

## Criterio mid

Un mid debe poder explicar las limitaciones de Database Mirroring frente a Always On AG.

## Criterio senior

Un senior debe planificar y ejecutar la migracion de sistemas legacy desde Database Mirroring hacia Always On AG.

## Errores comunes

- Implementar Database Mirroring en un sistema nuevo, ignorando que esta deprecated desde hace mas de una decada.
- No reconocer Database Mirroring en un sistema legacy durante una auditoria, dejando pasar una modernizacion necesaria.
- Asumir que la migracion de Mirroring a Always On AG es automatica o trivial, sin planificarla adecuadamente.
- No comunicar a stakeholders por que una tecnologia "que funciona" necesita ser reemplazada.

## Practica

- [ ] Investiga las limitaciones especificas de Database Mirroring frente a Always On AG.
- [ ] Investiga como identificar si un sistema en produccion usa Database Mirroring.
- [ ] Diseña un plan de migracion conceptual desde Mirroring hacia Always On AG.
- [ ] Compara la evolucion historica de alta disponibilidad en SQL Server con la de Postgres.
- [ ] Repasa todo el capitulo: AG, FCI, Log Shipping, Replication, y Mirroring, y en que escenario usarias cada uno.

## Preguntas de repaso

- Por que Database Mirroring esta deprecated, y que lo reemplazo?
- Que limitacion especifica tenia Mirroring que Always On AG resuelve?
- Como identificarias si un sistema legacy esta usando Database Mirroring?
- De las cinco tecnicas de este capitulo (AG, FCI, Log Shipping, Replication, Mirroring), cual elegirias para cada uno de estos escenarios: alta disponibilidad local con failover automatico, disaster recovery economico con RPO de minutos, y distribucion de un subconjunto de datos hacia un data warehouse?

## Cierre del capitulo

Con este tema cierras el capitulo de alta disponibilidad de SQL Server, habiendo cubierto el espectro completo de soluciones: desde Always On AG (la opcion moderna recomendada) hasta tecnologias legacy que aun podes encontrar en sistemas de produccion antiguos, todas conectadas con sus equivalentes ya estudiados en el libro de Postgres.
