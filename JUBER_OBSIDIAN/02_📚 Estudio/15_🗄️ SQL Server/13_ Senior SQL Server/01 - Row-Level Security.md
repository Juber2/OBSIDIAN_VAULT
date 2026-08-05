---
tags:
  - SQLServer
  - SeniorSQLServer
  - SQLServer-Senior
Creado: 2026-07-29
Relacionado:
  - Senior SQL Server
  - SQL SERVER
---

# 01 - Row-Level Security

## Que vas a aprender

En este capitulo vas a aprender **Row-Level Security (RLS)** de SQL Server, funcionalmente equivalente a Row Level Security de Postgres ya estudiada en profundidad en ese libro: restringir que filas puede ver/modificar cada usuario, aplicado directamente al servidor sin logica adicional en la aplicacion.

## Explicacion

RLS en SQL Server se implementa con una **funcion inline de seguridad** (`CREATE FUNCTION` que retorna 1 si la fila es visible para el usuario actual, tipicamente comparando una columna como `TenantId` contra `USER_NAME()` o un contexto de sesion) y una **Security Policy** (`CREATE SECURITY POLICY`) que aplica esa funcion como **filter predicate** (oculta filas en `SELECT`/`UPDATE`/`DELETE`) o **block predicate** (impide `INSERT`/`UPDATE` de filas que violarian la regla), exactamente el mismo modelo conceptual de policies ya dominado en Postgres, con sintaxis distinta.

La ventaja central es identica a la ya explicada en Postgres: la seguridad se aplica a nivel del motor de base de datos, protegiendo automaticamente incluso si una query nueva olvida agregar el filtro `WHERE TenantId = @actual` manualmente, cerrando una clase entera de vulnerabilidades de fuga de datos entre tenants en sistemas multi-tenant.

## Modelo mental

Pensa RLS de SQL Server exactamente igual que Row Level Security de Postgres ya dominada: un guardia de seguridad invisible parado en la puerta de cada fila, que automaticamente oculta o bloquea el acceso segun quien esta preguntando, sin que cada aplicacion tenga que acordarse de preguntarle "esto es mio?" en cada query.

## Ejemplo base

```sql
-- Funcion de seguridad: retorna 1 si la fila pertenece al tenant actual
CREATE FUNCTION dbo.fn_SeguridadTenant(@TenantId INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS resultado
WHERE @TenantId = CAST(SESSION_CONTEXT(N'TenantId') AS INT);

-- Security Policy: aplica la funcion como filter predicate
CREATE SECURITY POLICY PoliticaTenant
ADD FILTER PREDICATE dbo.fn_SeguridadTenant(TenantId) ON dbo.Cursos,
ADD BLOCK PREDICATE dbo.fn_SeguridadTenant(TenantId) ON dbo.Cursos AFTER INSERT
WITH (STATE = ON);

-- Establecer el contexto de sesion al conectar (desde la aplicacion)
EXEC sp_set_session_context @key = N'TenantId', @value = 5;
```

## Como se usa en un proyecto real

1. Se implementa RLS en un sistema multi-tenant (aplicable si StaffTrainingPlatform evoluciona a servir multiples organizaciones/clientes independientes) para garantizar aislamiento de datos a nivel del motor, no solo en la logica de la aplicacion.
2. Se establece el contexto de sesion (`sp_set_session_context`) inmediatamente al abrir cada conexion desde la aplicacion, antes de ejecutar cualquier query de negocio.
3. Se usa `BLOCK PREDICATE` ademas de `FILTER PREDICATE` para prevenir que un usuario inserte accidentalmente datos que pertenecerian a otro tenant, no solo ocultar datos ajenos en lecturas.
4. Se conecta esto directamente con Row Level Security de Postgres, aplicando el mismo modelo mental con sintaxis especifica de SQL Server.
5. Se documenta la Security Policy como parte critica de la arquitectura de seguridad, dado que un error en la funcion de seguridad podria exponer o bloquear datos incorrectamente para todo el sistema.

## Criterio junior

Un junior debe entender el proposito de RLS y poder leer una Security Policy existente.

## Criterio mid

Un mid debe implementar una funcion de seguridad y Security Policy basica para un escenario multi-tenant simple.

## Criterio senior

Un senior debe diseñar la estrategia completa de RLS para un sistema multi-tenant real, incluyendo el establecimiento seguro del contexto de sesion y testing exhaustivo de los predicados.

## Errores comunes

- No establecer el contexto de sesion correctamente en cada conexion, causando que RLS no filtre nada (o bloquee todo) inesperadamente.
- Usar solo `FILTER PREDICATE` sin `BLOCK PREDICATE`, permitiendo insertar datos que despues quedan invisibles pero pertenecen incorrectamente a otro tenant.
- No testear exhaustivamente la funcion de seguridad, dado que un error ahi puede exponer datos entre tenants silenciosamente.
- Confiar unicamente en filtros a nivel de aplicacion sin RLS, dejando la puerta abierta a fugas de datos si una query nueva olvida el filtro manual.

## Practica

- [ ] Implementa una funcion de seguridad y Security Policy basica para un escenario multi-tenant de practica.
- [ ] Verifica que `sp_set_session_context` efectivamente filtra los resultados segun el tenant activo.
- [ ] Prueba que `BLOCK PREDICATE` previene un `INSERT` que violaria la regla de seguridad.
- [ ] Compara esta sintaxis con Row Level Security de Postgres, identificando las equivalencias.
- [ ] Diseña la estrategia de RLS para un sistema multi-tenant hipotetico basado en StaffTrainingPlatform.

## Preguntas de repaso

- Que diferencia hay entre `FILTER PREDICATE` y `BLOCK PREDICATE`?
- Por que es critico establecer el contexto de sesion antes de ejecutar cualquier query de negocio?
- Que riesgo de seguridad cierra RLS que un filtro manual a nivel de aplicacion no garantiza?
- Como se compara este mecanismo con Row Level Security de Postgres?
