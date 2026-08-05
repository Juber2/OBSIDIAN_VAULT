---
tags:
  - NestJS
  - TypeScript
  - Nest-Arquitectura
Creado: 2026-07-27
Relacionado:
  - Arquitectura NestJS
  - NESTJS
---

# 03 - CQRS con NestJS

## Que vas a aprender

En este capitulo vas a aprender a aplicar **CQRS** (Command Query Responsibility Segregation) usando el modulo oficial `@nestjs/cqrs`: separar comandos (escritura) de queries (lectura), y opcionalmente incorporar event sourcing basico.

## Explicacion

CQRS propone separar el modelo usado para escribir datos del modelo usado para leerlos. En vez de un unico servicio con metodos `create`, `update`, `findAll`, se definen **Commands** (`CreateOrderCommand`) manejados por **CommandHandlers**, y **Queries** (`GetOrderQuery`) manejados por **QueryHandlers**, cada uno optimizado para su proposito.

Esto tiene sentido cuando lectura y escritura tienen necesidades muy distintas: escritura necesita validar invariantes de negocio complejas, lectura necesita ser rapida y puede usar una vista desnormalizada o incluso una base de datos distinta optimizada para consultas. Para un CRUD simple, CQRS agrega complejidad sin beneficio proporcional.

## Modelo mental

Pensa CQRS como separar la cocina (donde se preparan los platos, con reglas estrictas de calidad e higiene) del menu impreso (una vista simplificada y rapida de leer para el cliente). No usas la misma estructura para cocinar que para mostrar el menu, aunque ambos hablan del mismo negocio.

## Ejemplo base

```typescript
export class CreateOrderCommand {
  constructor(public readonly dto: CreateOrderDto) {}
}

@CommandHandler(CreateOrderCommand)
export class CreateOrderHandler implements ICommandHandler<CreateOrderCommand> {
  constructor(private ordersRepo: OrdersRepository, private eventBus: EventBus) {}

  async execute(command: CreateOrderCommand) {
    const order = await this.ordersRepo.save(command.dto);
    this.eventBus.publish(new OrderCreatedEvent(order.id));
    return order;
  }
}

// Controller
@Post()
create(@Body() dto: CreateOrderDto) {
  return this.commandBus.execute(new CreateOrderCommand(dto));
}
```

## Como se usa en un proyecto real

1. Se aplica CQRS solo en modulos con complejidad real de negocio en escritura, o con necesidades de lectura muy distintas a las de escritura.
2. Los Commands representan intencion de negocio (`ApproveOrderCommand`), no simples operaciones CRUD (`UpdateOrderCommand` generico).
3. Las Queries pueden leer de una proyeccion optimizada, distinta del modelo transaccional usado para escribir.
4. Se combina con eventos de dominio (`EventBus`) para desacoplar efectos posteriores a un comando exitoso.
5. Se documenta explicitamente por que un modulo usa CQRS y otros no, para que el equipo entienda la inconsistencia intencional.

## Criterio junior

Un junior debe poder implementar un Command simple con su handler y llamarlo desde un controller via `CommandBus`.

## Criterio mid

Un mid debe decidir en que modulos CQRS aporta valor real, e implementar Queries separadas de Commands cuando las necesidades de lectura y escritura difieren.

## Criterio senior

Un senior debe evaluar el costo/beneficio de CQRS (y de event sourcing, si se considera) para cada modulo del sistema, evitando aplicarlo de forma dogmatica donde un servicio simple resolveria el problema con menos complejidad.

## Errores comunes

- Aplicar CQRS a todo el sistema por moda, incluso en CRUDs simples sin justificacion de complejidad.
- Crear Commands que son simplemente wrappers de un metodo CRUD, sin capturar intencion real de negocio.
- No mantener consistencia entre el modelo de escritura y las proyecciones de lectura, generando datos desincronizados.
- Mezclar CQRS con la capa de controllers de forma inconsistente en distintos modulos del mismo proyecto sin documentar el criterio.

## Practica

- [ ] Implementa un Command simple con su handler y su llamada desde un controller.
- [ ] Implementa una Query separada para el mismo dominio del Command anterior.
- [ ] Evalua un modulo real de tu proyecto: justifica CQRS o no.
- [ ] Combina un Command con un evento de dominio publicado tras su ejecucion exitosa.
- [ ] Anota los criterios que usarias para decidir "este modulo usa CQRS, este no".

## Preguntas de repaso

- Que problema resuelve separar el modelo de lectura del modelo de escritura?
- Cuando NO conviene aplicar CQRS?
- Que diferencia hay entre un Command que representa intencion de negocio y uno que es solo un CRUD disfrazado?
- Como te asegurarias de que las proyecciones de lectura no queden desincronizadas del modelo de escritura?
