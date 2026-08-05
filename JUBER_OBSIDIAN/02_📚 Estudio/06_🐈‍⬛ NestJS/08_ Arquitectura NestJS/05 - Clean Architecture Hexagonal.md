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

# 05 - Clean Architecture Hexagonal

## Que vas a aprender

En este capitulo vas a aprender a aplicar **Clean Architecture / Arquitectura Hexagonal** en NestJS: puertos y adaptadores, inversion de dependencias hacia el dominio, y como estructurar carpetas para que el nucleo de negocio no dependa de NestJS ni de TypeORM.

## Explicacion

La idea central de la arquitectura hexagonal es que el **dominio y los casos de uso** (el "que hace" la aplicacion) no dependan de detalles de infraestructura (el "como" se implementa: HTTP, base de datos, colas). Esto se logra definiendo **puertos** (interfaces) que el dominio necesita, y **adaptadores** (implementaciones concretas) que la infraestructura provee, inyectados via DI (usando tokens custom, visto en el capitulo de Providers).

En NestJS esto significa que las entidades de dominio y los casos de uso viven en carpetas sin ningun import de `@nestjs/*` ni de TypeORM; los controllers, los repositorios TypeORM, los gateways de mensajeria, son todos "adaptadores" que implementan puertos definidos por el dominio.

## Modelo mental

Pensa el hexagono como el nucleo de negocio en el centro, rodeado de puertos (enchufes estandarizados). Del lado de afuera, distintos adaptadores se enchufan a esos puertos: un controller HTTP, un consumidor de cola, un test automatizado. El nucleo nunca sabe ni le importa que adaptador esta conectado en cada momento.

## Ejemplo base

```typescript
// dominio: puerto (interfaz), sin ningun import de Nest ni TypeORM
export interface OrdersRepositoryPort {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
}

// caso de uso: depende solo del puerto, no de la implementacion
export class CreateOrderUseCase {
  constructor(private readonly ordersRepo: OrdersRepositoryPort) {}
  async execute(data: CreateOrderInput) {
    const order = Order.create(data);
    await this.ordersRepo.save(order);
    return order;
  }
}

// adaptador: implementacion concreta con TypeORM
export const ORDERS_REPOSITORY = Symbol('ORDERS_REPOSITORY');

@Injectable()
export class TypeOrmOrdersRepository implements OrdersRepositoryPort {
  constructor(@InjectRepository(OrderEntity) private repo: Repository<OrderEntity>) {}
  async save(order: Order) { await this.repo.save(toEntity(order)); }
  async findById(id: string) { /* mapear entidad de persistencia a dominio */ return null; }
}
```

## Como se usa en un proyecto real

1. El dominio y los casos de uso se ubican en una carpeta separada (`domain/`, `application/`), sin ninguna dependencia de Nest o TypeORM.
2. Los puertos se definen en el dominio; los adaptadores (controllers, repositorios, clientes HTTP externos) viven en `infrastructure/` y se registran en el modulo Nest con tokens custom.
3. Se aplica esta arquitectura en modulos con reglas de negocio complejas y con expectativa de larga vida; no en CRUDs simples donde el costo de indireccion no se justifica.
4. Los casos de uso se testean sin ningun mock de infraestructura real, usando adaptadores fake en memoria.
5. Se documenta la regla de dependencia (las flechas de import siempre apuntan hacia el dominio, nunca al reves) para que el equipo la respete.

## Criterio junior

Un junior debe poder identificar, en un proyecto con esta arquitectura, que carpeta es dominio y cual es infraestructura, y respetar que el dominio no debe importar nada de Nest.

## Criterio mid

Un mid debe implementar un caso de uso completo con su puerto e implementacion, y testear el caso de uso con un adaptador fake sin infraestructura real.

## Criterio senior

Un senior debe decidir en que modulos del sistema esta arquitectura se justifica (complejidad de negocio real, necesidad de portabilidad de infraestructura) y en cuales seria sobre-ingenieria, y mantener la regla de dependencia auditada en el tiempo.

## Errores comunes

- Aplicar arquitectura hexagonal completa a un CRUD simple, multiplicando archivos e indireccion sin beneficio real.
- Dejar que un caso de uso importe algo de `@nestjs/*` o de TypeORM "solo por esta vez", rompiendo la regla de dependencia poco a poco.
- No testear los casos de uso con adaptadores fake, perdiendo la principal ventaja de esta arquitectura (testear el dominio sin infraestructura).
- Confundir esta arquitectura con una obligacion de usar microservicios; son decisiones independientes.

## Practica

- [ ] Define un puerto (interfaz) para un caso de uso de tu dominio.
- [ ] Implementa el caso de uso dependiendo solo del puerto, sin imports de Nest.
- [ ] Implementa un adaptador real (TypeORM) y un adaptador fake (en memoria) para el mismo puerto.
- [ ] Testea el caso de uso usando el adaptador fake, sin base de datos real.
- [ ] Evalua un modulo real de tu proyecto: justifica esta arquitectura o seria sobre-ingenieria.

## Preguntas de repaso

- Que es un puerto y que es un adaptador en arquitectura hexagonal?
- Hacia donde deben apuntar siempre las dependencias entre dominio e infraestructura?
- Que ventaja de testing da mantener el dominio libre de imports de Nest y TypeORM?
- En que tipo de modulo esta arquitectura seria sobre-ingenieria?
