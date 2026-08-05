---
tags:
  - NestJS
  - TypeScript
  - Nest-Datos
Creado: 2026-07-27
Relacionado:
  - TypeORM Prisma y Base de Datos
  - NESTJS
---

# 06 - Transacciones y Concurrencia

## Que vas a aprender

En este capitulo vas a aprender a manejar **transacciones** en NestJS (con TypeORM y Prisma), y a entender problemas de concurrencia como condiciones de carrera y bloqueos, con estrategias para evitarlos.

## Explicacion

Una transaccion agrupa varias operaciones de base de datos para que se apliquen todas o ninguna (atomicidad). Es indispensable en cualquier operacion que toque mas de una tabla y donde un fallo a mitad de camino dejaria datos inconsistentes (por ejemplo, descontar stock y crear una orden).

En TypeORM se usa `QueryRunner` con `startTransaction`/`commitTransaction`/`rollbackTransaction`, o el helper `dataSource.transaction()`. En Prisma se usa `$transaction()`, que soporta tanto un array de operaciones como una funcion interactiva. Ademas de la atomicidad, hay que pensar en **concurrencia**: dos requests que intentan modificar el mismo registro al mismo tiempo pueden generar condiciones de carrera, que se resuelven con locks pesimistas, locks optimistas (columna de version) o constraints a nivel de base de datos.

## Modelo mental

Pensa una transaccion como una promesa de todo-o-nada: si algo falla en el medio, la base de datos vuelve exactamente al estado anterior, como si nada hubiera pasado. La concurrencia es el problema aparte de "que pasa si dos personas hacen esa promesa al mismo tiempo sobre el mismo dato".

## Ejemplo base

```typescript
async transferir(fromId: string, toId: string, monto: number) {
  return this.dataSource.transaction(async (manager) => {
    const from = await manager.findOneOrFail(Cuenta, { where: { id: fromId }, lock: { mode: 'pessimistic_write' } });
    if (from.saldo < monto) throw new BadRequestException('Saldo insuficiente');

    await manager.decrement(Cuenta, { id: fromId }, 'saldo', monto);
    await manager.increment(Cuenta, { id: toId }, 'saldo', monto);
  });
}
```

## Como se usa en un proyecto real

1. Toda operacion que modifica mas de una tabla y debe ser atomica se envuelve en una transaccion.
2. Se usa lock pesimista (`pessimistic_write`) cuando la contencion es alta y hay que serializar el acceso; lock optimista (columna `version`) cuando la contencion es baja y se prefiere reintentar en caso de conflicto.
3. Las transacciones se mantienen lo mas cortas posible para no retener locks mas tiempo del necesario.
4. Se testean escenarios de concurrencia con tests que simulan requests simultaneos sobre el mismo recurso.
5. Se usan constraints de base de datos (unique, check) como ultima linea de defensa, incluso si la logica de aplicacion ya valida lo mismo.

## Criterio junior

Un junior debe entender que es una transaccion, cuando usarla, y poder envolver una operacion multi-tabla simple en una transaccion basica.

## Criterio mid

Un mid debe elegir entre lock optimista y pesimista segun el nivel de contencion esperado, y evitar transacciones largas que retengan locks innecesariamente.

## Criterio senior

Un senior debe diseñar la estrategia de concurrencia de operaciones criticas del negocio (pagos, inventario, reservas), definir niveles de aislamiento de transaccion cuando corresponda, y anticipar deadlocks en operaciones que tocan multiples recursos en orden variable.

## Errores comunes

- Modificar varias tablas relacionadas sin transaccion, dejando datos inconsistentes ante un fallo parcial.
- Mantener transacciones abiertas demasiado tiempo (por ejemplo, incluyendo llamadas HTTP externas dentro), bloqueando recursos innecesariamente.
- No manejar condiciones de carrera en operaciones de alta contencion (stock, saldos), permitiendo overselling o balances incorrectos.
- Ignorar el orden de adquisicion de locks entre operaciones, generando deadlocks intermitentes dificiles de reproducir.

## Practica

- [ ] Implementa una operacion multi-tabla envuelta en una transaccion.
- [ ] Simula un fallo a mitad de la transaccion y verifica el rollback.
- [ ] Implementa lock optimista con una columna de version en una entidad.
- [ ] Escribe un test que dispare dos operaciones concurrentes sobre el mismo recurso.
- [ ] Anota que operaciones de tu dominio necesitan transaccion y cuales no.

## Preguntas de repaso

- Que garantiza una transaccion que una serie de operaciones sueltas no garantiza?
- Cuando conviene lock optimista sobre lock pesimista?
- Que riesgo tiene mantener una transaccion abierta con una llamada HTTP externa adentro?
- Como podria surgir un deadlock entre dos transacciones concurrentes?
