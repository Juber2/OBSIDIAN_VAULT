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

# 03 - Relaciones TypeORM

## Que vas a aprender

En este capitulo vas a dominar las relaciones entre entidades en TypeORM: `@OneToMany`, `@ManyToOne`, `@ManyToMany`, `@OneToOne`, la diferencia entre carga **eager** y **lazy**, y como evitar el problema de N+1 queries.

## Explicacion

TypeORM modela relaciones de forma declarativa con decoradores en ambos lados de la relacion. El lado "dueño" de la relacion (donde vive la foreign key) se marca con `@JoinColumn()`. Por default las relaciones son **lazy**: no se cargan a menos que se pidan explicitamente con `relations` en la query o `.leftJoinAndSelect()` en el QueryBuilder.

El error mas comun en cualquier ORM (no solo TypeORM) es el problema de **N+1 queries**: pedir una lista de N registros y despues, por cada uno, hacer una query adicional para traer su relacion, terminando en N+1 queries en vez de 1 o 2. Esto se resuelve cargando relaciones explicitamente en la query original.

## Modelo mental

Pensa las relaciones como puentes entre tablas que TypeORM no cruza automaticamente a menos que se lo pidas. Si iterás una lista y cruzas el puente uno por uno dentro del loop, estas construyendo el puente N veces en vez de una sola.

## Ejemplo base

```typescript
@Entity()
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => User, (user) => user.orders)
  user: User;

  @OneToMany(() => OrderItem, (item) => item.order, { cascade: true })
  items: OrderItem[];
}

// Mal: N+1 -> una query por cada orden para traer su usuario
const orders = await this.ordersRepo.find();
for (const order of orders) {
  console.log(order.user.email); // dispara una query por cada order
}

// Bien: una sola query con join
const orders = await this.ordersRepo.find({ relations: { user: true, items: true } });
```

## Como se usa en un proyecto real

1. Las relaciones se declaran lazy por default; se cargan explicitamente segun lo que cada endpoint realmente necesita.
2. Se usa `relations` (o `leftJoinAndSelect` con QueryBuilder) para evitar N+1 en listados.
3. Se define claramente el lado dueño de cada relacion (`@JoinColumn`) para controlar donde vive la foreign key.
4. Se usa `cascade` con cuidado: solo donde el ciclo de vida de las entidades hijas realmente depende del padre.
5. Se revisan los logs de queries en desarrollo (`logging: true`) para detectar N+1 antes de que lleguen a produccion.

## Criterio junior

Un junior debe poder declarar relaciones basicas entre dos entidades y cargar una relacion explicitamente al hacer una consulta.

## Criterio mid

Un mid debe detectar y resolver problemas de N+1 en codigo existente, y elegir correctamente entre `find` con `relations` y `QueryBuilder` segun la complejidad de la consulta.

## Criterio senior

Un senior debe revisar en code review cualquier acceso a una relacion dentro de un loop, definir convenciones de carga de relaciones por endpoint, y decidir cuando desnormalizar datos en vez de depender de joins costosos.

## Errores comunes

- Acceder a una relacion lazy dentro de un `for`/`map` sin haberla cargado antes, generando N+1.
- Usar `cascade: true` sin entender que tambien puede borrar o modificar entidades relacionadas sin querer.
- Cargar relaciones que el endpoint no necesita, trayendo mas datos de los que se van a usar.
- No revisar los logs de SQL generado durante el desarrollo, dejando pasar queries ineficientes a produccion.

## Practica

- [ ] Modela dos entidades relacionadas con `@OneToMany`/`@ManyToOne`.
- [ ] Provoca un N+1 a proposito y observa las queries generadas con `logging: true`.
- [ ] Corrige ese N+1 usando `relations` o `QueryBuilder`.
- [ ] Revisa un caso donde `cascade` podria borrar datos sin que sea la intencion.
- [ ] Anota una checklist propia para revisar relaciones antes de mergear un PR.

## Preguntas de repaso

- Que es el problema de N+1 y por que aparece con relaciones lazy?
- Como cargarias explicitamente una relacion en una query de listado?
- Que riesgo tiene usar `cascade: true` sin analizarlo?
- Como detectarias N+1 antes de que llegue a produccion?
