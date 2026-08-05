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

# 05 - Deadlocks y su Prevencion

## Que vas a aprender

En este capitulo vas a cerrar el capitulo consolidando estrategias practicas de **prevencion de deadlocks**: orden consistente de acceso a recursos, transacciones cortas, y patrones de diseño que minimizan el riesgo de contencion en aplicaciones reales conectadas via NestJS/TypeORM.

## Explicacion

Un deadlock ocurre cuando dos (o mas) transacciones se bloquean mutuamente: la transaccion A tiene un lock que la transaccion B necesita, y B tiene un lock que A necesita, sin que ninguna pueda avanzar. Postgres detecta esta situacion automaticamente (revisando periodicamente el grafo de espera de locks) y la resuelve **abortando una de las dos transacciones** (eligiendo generalmente la que menos trabajo tiene invertido), devolviendo un error que la aplicacion debe capturar y manejar, tipicamente reintentando la transaccion completa desde el principio.

La prevencion mas efectiva es de **diseño**, no reactiva: mantener un **orden consistente** de acceso a recursos en todo el codigo (si siempre se bloquea la tabla "cuentas" antes que "transferencias", nunca al reves, un deadlock entre esos dos recursos se vuelve imposible por diseño), mantener transacciones **cortas** (menos tiempo con locks abiertos significa menos oportunidad de contencion), y evitar operaciones lentas (llamadas HTTP externas, procesamiento pesado) dentro de una transaccion con locks activos.

## Modelo mental

Pensa la prevencion de deadlocks por orden consistente como una regla de transito de un unico sentido en un cruce complicado: si todos los autos siempre entran por el mismo lado en el mismo orden, nunca pueden quedar frente a frente bloqueandose mutuamente, sin importar cuantos autos haya circulando simultaneamente.

## Ejemplo base

```typescript
// Patron de orden consistente en NestJS/TypeORM: siempre bloquear en el mismo orden
// (por ejemplo, siempre por ID ascendente, sin importar el orden de los parametros de entrada)
async transferir(cuentaAId: number, cuentaBId: number, monto: number) {
  const [primeraId, segundaId] = [cuentaAId, cuentaBId].sort((a, b) => a - b);

  return this.dataSource.transaction(async (manager) => {
    const primera = await manager.findOneOrFail(Cuenta, {
      where: { id: primeraId }, lock: { mode: 'pessimistic_write' },
    });
    const segunda = await manager.findOneOrFail(Cuenta, {
      where: { id: segundaId }, lock: { mode: 'pessimistic_write' },
    });
    // logica de transferencia usando primera/segunda segun corresponda...
  });
}
```

## Como se usa en un proyecto real

1. Se establece una convencion de equipo de orden consistente de bloqueo (por ejemplo, siempre por ID ascendente) para cualquier operacion que bloquee mas de un recurso.
2. Se mantienen las transacciones lo mas cortas posible, moviendo cualquier logica que no necesite estar dentro del lock (validaciones que no dependen de datos bloqueados, llamadas externas) fuera de la transaccion.
3. Se implementa logica de reintento generica (con backoff, similar al patron ya visto en el libro de NestJS para conexiones) para transacciones que fallan por deadlock detectado.
4. Se monitorea la tasa de deadlocks en produccion (`pg_stat_database.deadlocks`) como señal de que el diseño de concurrencia necesita revision si aparecen con frecuencia inusual.
5. Se conecta esto con el capitulo de Transacciones y Concurrencia del libro de NestJS, cerrando el circulo entre la teoria de locks/deadlocks y su implementacion practica con TypeORM.

## Criterio junior

Un junior debe entender que es un deadlock y que Postgres lo detecta y resuelve automaticamente abortando una transaccion.

## Criterio mid

Un mid debe implementar orden consistente de bloqueo en operaciones que afectan multiples recursos, y logica de reintento ante deadlocks detectados.

## Criterio senior

Un senior debe monitorear la tasa de deadlocks en produccion como metrica de salud del diseño de concurrencia, y liderar el rediseño de patrones de acceso a datos que generan contencion frecuente.

## Errores comunes

- No establecer un orden consistente de bloqueo entre distintas operaciones que acceden a los mismos recursos, generando deadlocks intermitentes dificiles de reproducir en desarrollo.
- Mantener transacciones abiertas mas tiempo del necesario, incluyendo logica que no requiere estar dentro del lock.
- No implementar reintentos ante deadlocks, dejando que fallen visiblemente para el usuario en vez de resolverse automaticamente de forma transparente.
- No monitorear la tasa de deadlocks en produccion, perdiendo la señal temprana de un problema de diseño de concurrencia que empeora con el crecimiento del sistema.

## Practica

- [ ] Diseña (o identifica en tu codigo real) un caso donde dos operaciones podrian bloquear recursos en orden distinto, y corrigelo con orden consistente.
- [ ] Implementa logica de reintento generica para deadlocks en un servicio de practica.
- [ ] Reduce el alcance de una transaccion existente, moviendo logica innecesaria fuera del bloque transaccional.
- [ ] Investiga como consultarias la tasa de deadlocks de tu base de datos en produccion.
- [ ] Documenta la convencion de orden de bloqueo que estableceria para tu equipo.

## Preguntas de repaso

- Como decide Postgres cual de dos transacciones en deadlock abortar?
- Por que un orden consistente de acceso a recursos previene deadlocks por diseño?
- Que beneficio da mantener las transacciones lo mas cortas posible en relacion al riesgo de deadlocks?
- Como monitorearias la salud del diseño de concurrencia de un sistema en produccion?
