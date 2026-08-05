---
tags:
  - PostgreSQL
  - Transacciones
  - Postgres-Transacciones
Creado: 2026-07-29
Relacionado:
  - Transacciones y Concurrencia
  - POSTGRESQL
---

# 03 - Niveles de Aislamiento

## Que vas a aprender

En este capitulo vas a aprender los **niveles de aislamiento** de transacciones que soporta Postgres, y los fenomenos de concurrencia que cada uno previene o permite: **Read Committed** (default), **Repeatable Read**, y **Serializable**.

## Explicacion

El nivel de aislamiento controla cuanto "ve" una transaccion de los cambios hechos por otras transacciones concurrentes. **Read Committed** (el default de Postgres) garantiza que cada consulta individual dentro de una transaccion solo ve datos ya confirmados (`COMMIT`) por otras transacciones, pero distintas consultas dentro de la **misma** transaccion pueden ver datos distintos si otra transaccion confirmo cambios entre medio (fenomeno llamado **non-repeatable read**).

**Repeatable Read** garantiza que toda la transaccion ve una "fotografia" consistente del estado de la base tomada al inicio, sin importar que otras transacciones confirmen cambios mientras tanto, previniendo non-repeatable reads. **Serializable** (el mas estricto) garantiza que el resultado de ejecutar transacciones concurrentes es equivalente a haberlas ejecutado una despues de la otra en algun orden, previniendo tambien **anomalias de escritura sesgada** (phantom reads y write skew), a costa de que Postgres puede rechazar una transaccion con un error de serializacion que la aplicacion debe estar preparada para reintentar.

## Modelo mental

Pensa Read Committed como leer un documento compartido en tiempo real: cada vez que mirás una seccion, ves la version mas reciente confirmada, aunque eso signifique que dos miradas distintas a la misma seccion, minutos aparte, muestren contenido diferente si alguien lo edito mientras tanto. Repeatable Read es como tomar una fotografia completa del documento al empezar a leer, y seguir mirando esa misma fotografia durante toda tu lectura, sin importar que el documento real siga cambiando en paralelo. Serializable es como si el sistema garantizara que, pase lo que pase, el resultado final sera exactamente el mismo que si cada persona hubiera leido y editado el documento una despues de la otra, en fila, nunca al mismo tiempo.

## Ejemplo base

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT saldo FROM cuentas WHERE id = 1;   -- ve X
-- (otra transaccion confirma un cambio al saldo mientras tanto)
SELECT saldo FROM cuentas WHERE id = 1;   -- SIGUE viendo X, no el nuevo valor
COMMIT;

BEGIN ISOLATION LEVEL SERIALIZABLE;
-- operaciones...
COMMIT;
-- Si Postgres detecta una anomalia de serializacion, este COMMIT falla:
-- ERROR: could not serialize access due to concurrent update
-- la aplicacion debe capturar este error y reintentar la transaccion completa
```

## Como se usa en un proyecto real

1. Se usa Read Committed (el default) para la gran mayoria de operaciones, dado que es suficiente para la mayoria de casos y tiene el menor overhead.
2. Se usa Repeatable Read cuando una transaccion necesita ver un estado consistente durante toda su duracion, como al generar un reporte que hace multiples consultas relacionadas.
3. Se usa Serializable para operaciones criticas de negocio donde incluso anomalias sutiles de concurrencia (como write skew) serian inaceptables, como transferencias financieras complejas.
4. Se implementa logica de reintento en la aplicacion para transacciones Serializable, dado que Postgres puede rechazarlas legitimamente ante conflictos de concurrencia detectados.
5. Se conecta esto con el capitulo de Transacciones y Concurrencia del libro de NestJS, donde se vio el concepto de lock optimista/pesimista como estrategia complementaria a nivel de aplicacion.

## Criterio junior

Un junior debe conocer los tres niveles de aislamiento principales y entender que Read Committed es el default de Postgres.

## Criterio mid

Un mid debe elegir el nivel de aislamiento apropiado segun la sensibilidad de una operacion a anomalias de concurrencia, e implementar reintentos para transacciones Serializable.

## Criterio senior

Un senior debe diseñar la estrategia de aislamiento de operaciones criticas del sistema, balanceando la garantia de correccion contra el costo de performance y la complejidad de manejo de reintentos.

## Errores comunes

- No implementar logica de reintento para transacciones Serializable, dejando que el error de serializacion se propague como un fallo no manejado al usuario.
- Usar Serializable para toda operacion "por seguridad" sin considerar el costo de performance y complejidad adicional que no siempre se justifica.
- No entender que Read Committed permite que distintas consultas dentro de la misma transaccion vean datos distintos, causando bugs sutiles en logica que asume consistencia durante toda la transaccion.
- Confundir el nivel de aislamiento de transaccion con locks explicitos a nivel de fila, que son mecanismos complementarios pero distintos.

## Practica

- [ ] Simula el fenomeno de non-repeatable read con dos sesiones concurrentes en Read Committed.
- [ ] Repite el mismo experimento con Repeatable Read y observa la diferencia de comportamiento.
- [ ] Provoca un error de serializacion con dos transacciones Serializable concurrentes conflictivas.
- [ ] Implementa logica de reintento simple para manejar ese error de serializacion.
- [ ] Identifica una operacion de tu proyecto que se beneficiaria de un nivel de aislamiento mas estricto que el default.

## Preguntas de repaso

- Que fenomeno de concurrencia previene Repeatable Read que Read Committed no previene?
- Que garantia adicional da Serializable sobre Repeatable Read, y a que costo?
- Por que una aplicacion que usa Serializable debe estar preparada para reintentar transacciones?
- Cuando elegirias un nivel de aislamiento mas estricto que el default de Postgres?
