---
tags:
  - PostgreSQL
  - Senior
  - Postgres-Senior
Creado: 2026-07-29
Relacionado:
  - Senior PostgreSQL
  - POSTGRESQL
---

# 02 - Seguridad Avanzada Row Level Security

## Que vas a aprender

En este capitulo vas a aprender **Row Level Security (RLS)**, una feature de Postgres que aplica filtros de seguridad **automaticamente a nivel de fila**, complementando el modelo de multi-tenancy ya visto en el libro de NestJS con una capa de defensa en profundidad a nivel de base de datos.

## Explicacion

Ya se vio en el libro de NestJS que el aislamiento multi-tenant por columna (`tenantId`) requiere que **cada query** filtre correctamente por esa columna, dependiendo de que el desarrollador nunca lo olvide (un riesgo real de error humano). **RLS** resuelve esto a nivel de motor: una vez activado en una tabla (`ALTER TABLE ... ENABLE ROW LEVEL SECURITY`) y definida una **politica** (`CREATE POLICY`), Postgres **automaticamente** filtra cualquier consulta sobre esa tabla segun la politica, sin importar si el desarrollador que escribio esa query especifica se acordo de agregar el filtro manualmente o no.

Esto es exactamente el mismo principio de "defensa en profundidad" ya visto en el capitulo de Seguridad Senior del libro de NestJS: RLS no reemplaza la logica de autorizacion de la aplicacion, la **complementa** como una ultima linea de defensa a nivel de base de datos que actua incluso si un desarrollador olvida un filtro, o si alguien accede directamente a la base de datos sin pasar por la API.

## Modelo mental

Pensa RLS como un filtro de seguridad instalado directamente en cada puerta de un archivo compartido, en vez de confiar en que cada empleado (cada query) recuerde revisar manualmente su credencial antes de sacar un documento. Con RLS, incluso si un empleado nuevo o distraido intenta acceder sin verificar, la puerta misma ya se encarga de dejarlo pasar solo hacia los documentos que le corresponden.

## Ejemplo base

```sql
ALTER TABLE pedidos ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON pedidos
    USING (tenant_id = current_setting('app.current_tenant')::bigint);

-- La aplicacion establece el tenant actual al inicio de cada conexion/transaccion
SET app.current_tenant = '42';

SELECT * FROM pedidos;   -- automaticamente filtrado a tenant_id = 42, SIN filtro manual explicito

-- Un intento de acceder a otro tenant simplemente no devuelve nada, sin error
SELECT * FROM pedidos WHERE tenant_id = 99;   -- vacio, RLS lo bloquea silenciosamente
```

## Como se usa en un proyecto real

1. Se activa RLS en tablas multi-tenant como capa adicional de defensa en profundidad, complementando (no reemplazando) el filtrado explicito ya presente en el codigo de la aplicacion.
2. Se establece el contexto del tenant actual (`SET app.current_tenant`) al inicio de cada conexion o transaccion desde la aplicacion NestJS, tipicamente en un interceptor o middleware.
3. Se documenta claramente que tablas tienen RLS activo, dado que su comportamiento no es visible simplemente leyendo el codigo de la aplicacion (similar al riesgo ya visto con triggers en el capitulo de Funciones).
4. Se testean politicas de RLS con casos especificos, verificando que un tenant efectivamente no puede ver datos de otro incluso con una query directa sin filtro.
5. Se conecta esto directamente con el capitulo de multi-tenancy del libro de NestJS, cerrando el circulo entre la estrategia de aislamiento a nivel de aplicacion y su refuerzo a nivel de base de datos.

## Criterio junior

Un junior debe entender conceptualmente que RLS filtra automaticamente filas segun una politica, sin necesitar que cada query lo haga manualmente.

## Criterio mid

Un mid debe activar RLS en una tabla multi-tenant y definir una politica basica, integrandola con el contexto de tenant establecido desde la aplicacion.

## Criterio senior

Un senior debe decidir cuando RLS se justifica como capa adicional de defensa en profundidad para datos sensibles multi-tenant, y auditar que las politicas estan correctamente implementadas y testeadas.

## Errores comunes

- No activar RLS en tablas multi-tenant sensibles, confiando exclusivamente en que el codigo de la aplicacion nunca olvidara el filtro manual.
- No establecer correctamente el contexto de tenant (`SET app.current_tenant`) en cada conexion, causando que RLS bloquee todo o filtre incorrectamente.
- No documentar que tablas tienen RLS activo, generando comportamiento "misterioso" para quien no sabe que existe esa politica.
- No testear politicas de RLS con casos especificos que verifiquen el aislamiento real entre tenants.

## Practica

- [ ] Activa RLS en una tabla multi-tenant de practica y define una politica basica de aislamiento.
- [ ] Establece el contexto de tenant con `SET` y verifica que las consultas se filtran automaticamente.
- [ ] Intenta acceder a datos de otro tenant con una query sin filtro manual y verifica que RLS lo bloquea.
- [ ] Documenta las tablas con RLS activo de un esquema de practica.
- [ ] Conecta este capitulo con la estrategia de multi-tenancy por columna ya vista en el libro de NestJS.

## Preguntas de repaso

- Que ventaja de seguridad da RLS sobre depender exclusivamente de filtros manuales en cada query de la aplicacion?
- Como se establece el contexto de tenant que una politica de RLS usa para filtrar?
- Por que RLS complementa, en vez de reemplazar, la logica de autorizacion de la aplicacion?
- Que verificarias al testear una politica de RLS para confirmar que realmente aisla los datos correctamente?
