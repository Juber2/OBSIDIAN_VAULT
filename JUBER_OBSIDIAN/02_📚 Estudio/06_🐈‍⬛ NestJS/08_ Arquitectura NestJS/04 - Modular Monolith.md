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

# 04 - Modular Monolith

## Que vas a aprender

En este capitulo vas a aprender a diseñar un **monolito modular**: una unica aplicacion desplegable, pero internamente dividida en modulos de dominio con limites estrictos, como paso intermedio realista entre "todo mezclado" y microservicios.

## Explicacion

Un monolito modular aprovecha la simplicidad operativa de desplegar una sola aplicacion (un solo proceso, una sola base de datos, un solo pipeline de deploy) mientras impone disciplina arquitectonica: cada modulo de dominio (`users`, `orders`, `billing`) tiene limites claros, no accede directamente a las tablas de otro modulo, y se comunica con otros modulos solo a traves de interfaces explicitas (servicios exportados o eventos de dominio).

Esta es la arquitectura recomendada para la gran mayoria de proyectos que "algun dia podrian necesitar microservicios": permite escalar el equipo (cada squad dueño de sus modulos) sin pagar el costo operativo de microservicios (redes, latencia, consistencia eventual, observabilidad distribuida) antes de necesitarlo realmente.

## Modelo mental

Pensa un monolito modular como un edificio de oficinas con paredes reales entre departamentos, pero todos bajo el mismo techo y la misma direccion: cada departamento tiene su propio proceso interno, pero para pedirle algo a otro departamento hay que pasar por su mostrador de atencion (la interfaz publica), nunca entrar caminando a revisar sus archivos.

## Ejemplo base

```typescript
// billing/billing.module.ts
@Module({
  providers: [BillingService],
  exports: [BillingService], // unica puerta de entrada al modulo
})
export class BillingModule {}

// orders/orders.service.ts
@Injectable()
export class OrdersService {
  constructor(private billingService: BillingService) {} // usa la interfaz publica

  async checkout(order: Order) {
    await this.billingService.charge(order.total); // nunca toca tablas de billing directo
  }
}
```

## Como se usa en un proyecto real

1. Cada modulo de dominio tiene sus propias entidades y nunca hace queries directas sobre tablas de otro modulo.
2. La comunicacion entre modulos pasa por servicios exportados (llamadas sincronas) o eventos de dominio (llamadas asincronas desacopladas).
3. Se documenta un mapa de dependencias permitidas entre modulos, evitando dependencias circulares de dominio.
4. Se disena cada modulo pensando "si mañana esto se separa en su propio servicio, que tan dificil seria", sin necesariamente hacerlo.
5. Se revisa en code review cualquier import que rompa el limite de un modulo (acceder a un repositorio de otro dominio directamente).

## Criterio junior

Un junior debe poder identificar cuando un modulo esta accediendo directamente a datos de otro modulo en vez de usar su interfaz publica.

## Criterio mid

Un mid debe diseñar la interfaz publica de un modulo de dominio (que expone, que no) y usar eventos de dominio para desacoplar comunicacion entre modulos donde no se necesita respuesta inmediata.

## Criterio senior

Un senior debe diseñar el mapa completo de modulos y sus dependencias permitidas para un sistema grande, decidir que modulos son candidatos reales a separarse en microservicios en el futuro, y establecer el proceso de revision arquitectonica que mantiene esos limites en el tiempo.

## Errores comunes

- Dejar que un modulo acceda directo a las tablas de otro, rompiendo el limite y generando acoplamiento oculto dificil de deshacer despues.
- Migrar prematuramente a microservicios sin haber probado primero la disciplina de un monolito modular, pagando el costo operativo antes de necesitarlo.
- No documentar el mapa de dependencias permitidas, dejando que cada desarrollador decida por su cuenta que puede importar de donde.
- Crear dependencias circulares entre modulos de dominio por mal diseño de limites desde el inicio.

## Practica

- [ ] Revisa un proyecto existente y detecta accesos directos entre modulos que deberian pasar por una interfaz publica.
- [ ] Diseña la interfaz publica de un modulo de dominio, decidiendo que exporta y que no.
- [ ] Reemplaza una llamada directa entre modulos por un evento de dominio donde tenga sentido.
- [ ] Dibuja el mapa de dependencias permitidas entre modulos de un proyecto real.
- [ ] Anota que modulo de tu sistema seria el primer candidato a separarse en microservicio, y por que.

## Preguntas de repaso

- Que ventaja operativa tiene un monolito modular sobre microservicios desde el dia uno?
- Como deberian comunicarse dos modulos de dominio dentro de un monolito modular?
- Que señal indica que un modulo esta listo para separarse en su propio microservicio?
- Por que documentar el mapa de dependencias permitidas ayuda a mantener la arquitectura en el tiempo?
