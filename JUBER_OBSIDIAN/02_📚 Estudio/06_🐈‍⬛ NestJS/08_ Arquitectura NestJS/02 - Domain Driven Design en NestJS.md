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

# 02 - Domain Driven Design en NestJS

## Que vas a aprender

En este capitulo vas a aprender a aplicar conceptos de **Domain Driven Design (DDD)** dentro de NestJS: entidades de dominio ricas (no anemicas), value objects, agregados, y como mantener el dominio libre de dependencias de framework.

## Explicacion

DDD propone modelar el software alrededor del lenguaje y las reglas reales del negocio, no alrededor de tablas de base de datos. Una entidad de dominio "rica" encapsula su propia logica de validacion e invariantes (por ejemplo, `Order.addItem()` valida internamente que la orden no este cerrada), a diferencia de una entidad "anemica" que es solo un contenedor de datos con toda la logica dispersa en servicios externos.

Un **agregado** es un grupo de entidades relacionadas que se tratan como una unidad de consistencia (por ejemplo, `Order` y sus `OrderItem` siempre se modifican juntos, a traves de la raiz del agregado). Un **value object** representa un concepto sin identidad propia (`Money`, `Email`), inmutable, cuya igualdad se basa en su valor, no en un ID.

## Modelo mental

Pensa el dominio como el nucleo puro del negocio, sin saber nada de HTTP, bases de datos ni frameworks: podria, en teoria, correr en un test sin ninguna infraestructura. Todo lo que es NestJS, TypeORM, Express, vive alrededor de ese nucleo, nunca adentro.

## Ejemplo base

```typescript
export class Email {
  private constructor(public readonly value: string) {}
  static create(value: string): Email {
    if (!/^[^@]+@[^@]+\.[^@]+$/.test(value)) throw new Error('Email invalido');
    return new Email(value);
  }
}

export class Order {
  private items: OrderItem[] = [];
  private status: 'open' | 'closed' = 'open';

  addItem(item: OrderItem) {
    if (this.status === 'closed') throw new Error('No se puede modificar una orden cerrada');
    this.items.push(item);
  }
}
```

## Como se usa en un proyecto real

1. Las entidades de dominio viven en una capa separada de las entidades de persistencia (TypeORM), aunque en proyectos chicos a veces se acepta unificarlas por pragmatismo.
2. Toda modificacion a un agregado pasa por metodos de la raiz del agregado, nunca modificando entidades hijas directamente desde afuera.
3. Los value objects se usan para conceptos que aparecen repetidamente con reglas propias (`Money`, `Email`, `Slug`), evitando repetir validacion dispersa.
4. El dominio no importa nada de `@nestjs/*` ni de TypeORM; los servicios de aplicacion (capa Nest) son los que orquestan el dominio con la infraestructura.
5. Se aplica DDD solo donde el dominio tiene complejidad real que lo justifique; un CRUD simple no necesita este nivel de modelado.

## Criterio junior

Un junior debe entender la diferencia entre una entidad anemica y una rica, y poder mover una validacion simple desde un servicio hacia el metodo de la entidad correspondiente.

## Criterio mid

Un mid debe modelar un agregado simple con su raiz controlando las modificaciones, y crear al menos un value object para un concepto que se repite en el dominio.

## Criterio senior

Un senior debe decidir cuando DDD aporta valor real y cuando es sobre-ingenieria para el tamaño del proyecto, definir los limites de los bounded contexts del sistema, y mantener el dominio realmente aislado de la infraestructura a medida que el proyecto crece.

## Errores comunes

- Aplicar DDD completo a un CRUD simple sin complejidad de negocio real, agregando capas sin beneficio.
- Dejar que el dominio importe decoradores de TypeORM o de Nest, acoplandolo a la infraestructura y perdiendo el beneficio de aislarlo.
- Modificar entidades hijas de un agregado directamente desde un servicio externo, saltandose las invariantes de la raiz.
- Confundir DDD tactico (entidades, value objects, agregados) con la necesidad de tener microservicios; son conceptos independientes.

## Practica

- [ ] Identifica una entidad anemica en un proyecto existente y mueve una validacion hacia un metodo propio de la entidad.
- [ ] Crea un value object para un concepto que se repite en tu dominio (dinero, email, un identificador con formato especial).
- [ ] Modela un agregado simple con su raiz controlando las modificaciones a sus hijos.
- [ ] Evalua si tu proyecto actual justifica DDD completo o si un modelo mas simple alcanza.
- [ ] Anota los bounded contexts que identificarias en un sistema real que conozcas.

## Preguntas de repaso

- Que diferencia hay entre una entidad anemica y una entidad rica?
- Que es un agregado y por que su raiz controla las modificaciones a sus hijos?
- Para que sirve un value object y en que se diferencia de una entidad?
- Cuando NO valdria la pena aplicar DDD a un proyecto?
