---
tags:
  - PostgreSQL
  - Cloud
  - Postgres-Cloud
Creado: 2026-07-29
Relacionado:
  - Postgres en la Nube y Contenedores
  - POSTGRESQL
---

# 02 - Servicios Gestionados RDS Cloud SQL

## Que vas a aprender

En este capitulo vas a aprender que servicios de Postgres **gestionados** (AWS RDS, Google Cloud SQL, Azure Database for PostgreSQL, y opciones nativas de Postgres como Supabase/Neon) ofrecen y delegan, y como esto cambia (o simplifica) todo lo aprendido en los capitulos de Administracion y Alta Disponibilidad.

## Explicacion

Un servicio gestionado toma la responsabilidad de gran parte de lo ya estudiado en este libro: backups automaticos (con PITR, ver capitulo de Administracion, ofrecido como feature integrada), parches de seguridad y actualizaciones de version aplicados por el proveedor, replicas de solo lectura configurables con un clic (ver capitulo de Alta Disponibilidad), y failover automatico (equivalente a lo que Patroni resuelve manualmente) gestionado por la plataforma. A cambio, se pierde acceso al sistema operativo subyacente y, en muchos casos, la posibilidad de instalar cualquier extension arbitraria (solo las que el proveedor soporta explicitamente).

Esta es la misma decision arquitectonica de "operar uno mismo versus usar gestionado" ya vista de forma general en el capitulo Senior del libro de Linux: para la mayoria de equipos sin experiencia profunda en administracion de bases de datos, un servicio gestionado reduce significativamente el riesgo operativo, aceptando el costo adicional de precio y la perdida de control granular sobre configuracion de bajo nivel.

## Modelo mental

Pensa un servicio gestionado como alquilar un departamento con administracion de edificio incluida: no tenes que preocuparte por el mantenimiento del ascensor, la seguridad del edificio, o las reparaciones de la caldera (backups, parches, failover), a cambio de un costo mensual mas alto y menos libertad de hacer modificaciones estructurales por tu cuenta (menos extensiones disponibles, menos configuracion de bajo nivel).

## Ejemplo base

```
Comparativa conceptual de responsabilidades:

| Tarea                          | Auto-gestionado | Servicio Gestionado |
|----------------------------------|-------------------|------------------------|
| Backups y PITR                   | Vos configuras    | Incluido, un clic      |
| Parches de seguridad del SO      | Vos aplicas       | Automatico             |
| Replicas de solo lectura         | Vos configuras    | Un clic                |
| Failover automatico              | Patroni, vos armas | Automatico             |
| Acceso al sistema operativo      | Total              | Ninguno                |
| Extensiones disponibles          | Cualquiera         | Solo las soportadas    |
| Costo                            | Menor (infra base) | Mayor (servicio premium)|
```

## Como se usa en un proyecto real

1. Se elige un servicio gestionado por defecto para la mayoria de equipos sin experiencia dedicada en administracion de Postgres, reduciendo riesgo operativo.
2. Se verifica la disponibilidad de extensiones necesarias (PostGIS, pg_trgm, ver capitulo de Tipos Avanzados) antes de comprometerse con un proveedor especifico.
3. Se evalua auto-gestionar (con o sin Patroni, ver capitulo anterior) solo cuando hay requisitos especificos que ningun servicio gestionado cubre, o cuando el costo a escala lo justifica con experiencia operativa real disponible.
4. Se sigue aplicando todo lo aprendido en el capitulo de Modelado, Indices, y Transacciones exactamente igual, dado que esos conocimientos son independientes de quien administra la infraestructura.
5. Se documenta esta decision como un ADR, aplicando el mismo marco de decisiones arquitectonicas visto en los libros de Docker y Linux.

## Criterio junior

Un junior debe entender que responsabilidades delega un servicio gestionado, y poder conectarse a uno desde una aplicacion NestJS.

## Criterio mid

Un mid debe verificar disponibilidad de extensiones necesarias en un proveedor especifico antes de comprometerse, y configurar replicas de solo lectura gestionadas.

## Criterio senior

Un senior debe decidir con criterio entre auto-gestionar y usar un servicio gestionado, documentando esa decision como un ADR con los tradeoffs considerados.

## Errores comunes

- Elegir un servicio gestionado sin verificar que soporta las extensiones necesarias para el proyecto, descubriendo la limitacion tarde en el desarrollo.
- Auto-gestionar Postgres en produccion sin la experiencia operativa real necesaria, subestimando la complejidad de backups, replicacion y failover correctos.
- No aplicar el conocimiento de modelado, indices y optimizacion de este libro solo porque "la nube se encarga de todo", cuando eso sigue siendo responsabilidad del desarrollador sin importar quien administra la infraestructura.
- No documentar la decision entre gestionado y auto-gestionado como una decision arquitectonica real, tratandola como una eleccion trivial sin justificacion.

## Practica

- [ ] Investiga las opciones de servicio gestionado de Postgres disponibles (RDS, Cloud SQL, Supabase, Neon) y compara sus features.
- [ ] Verifica si un proveedor especifico soporta una extension que necesites (PostGIS, pg_trgm).
- [ ] Configura una conexion desde una aplicacion NestJS de practica hacia un servicio gestionado (usando un tier gratuito si esta disponible).
- [ ] Documenta un ADR comparando auto-gestionar versus usar gestionado para un proyecto real tuyo.
- [ ] Anota que responsabilidades de este libro seguirian siendo tuyas incluso usando un servicio gestionado.

## Preguntas de repaso

- Que responsabilidades delega tipicamente un servicio gestionado de Postgres?
- Que se pierde en control y flexibilidad al usar un servicio gestionado?
- Que verificarias antes de comprometerte con un proveedor gestionado especifico?
- Que conocimientos de este libro siguen siendo responsabilidad tuya incluso usando un servicio gestionado?
