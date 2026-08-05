---
tags:
  - NestJS
  - TypeScript
  - Nest-Async
Creado: 2026-07-27
Relacionado:
  - Colas Eventos y Tiempo Real
  - NESTJS
---

# 01 - EventEmitter y Eventos de Dominio

## Que vas a aprender

En este capitulo vas a aprender a desacoplar efectos secundarios de la logica principal usando **eventos de dominio** con `@nestjs/event-emitter`, en vez de encadenar llamadas directas entre servicios.

## Explicacion

Cuando una accion de negocio dispara varios efectos secundarios (crear un usuario envia un email de bienvenida, notifica al equipo de ventas, actualiza una metrica), acoplar todo eso dentro del mismo metodo del servicio principal genera una clase que crece sin limite y que se vuelve fragil: si falla el envio de email, no deberia fallar la creacion del usuario.

`@nestjs/event-emitter` permite que el servicio principal solo emita un evento (`UserCreatedEvent`) sin saber quien lo escucha ni cuantos efectos secundarios dispara. Otros modulos se suscriben a ese evento con `@OnEvent()` de forma completamente desacoplada.

## Modelo mental

Pensa los eventos de dominio como un anuncio por altavoz: el servicio que crea el usuario simplemente anuncia "esto paso", sin saber ni importarle quien esta escuchando. Cualquier modulo interesado puede reaccionar sin que el emisor original necesite conocerlo.

## Ejemplo base

```typescript
export class UserCreatedEvent {
  constructor(public readonly userId: string, public readonly email: string) {}
}

@Injectable()
export class UsersService {
  constructor(private eventEmitter: EventEmitter2) {}

  async create(dto: CreateUserDto) {
    const user = await this.usersRepo.save(dto);
    this.eventEmitter.emit('user.created', new UserCreatedEvent(user.id, user.email));
    return user;
  }
}

@Injectable()
export class WelcomeEmailListener {
  @OnEvent('user.created')
  async handle(event: UserCreatedEvent) {
    await this.mailerService.sendWelcome(event.email);
  }
}
```

## Como se usa en un proyecto real

1. Se emiten eventos para efectos secundarios que no son parte del resultado principal de la operacion (notificaciones, metricas, integraciones).
2. Cada listener se mantiene enfocado en una sola responsabilidad, y falla de forma aislada sin afectar al emisor.
3. Se nombran los eventos en pasado (`user.created`, `order.paid`), reflejando que algo ya sucedio.
4. Se documenta el catalogo de eventos del sistema para que el equipo sepa que reacciones existen ante cada evento.
5. Para efectos secundarios que requieren garantia de entrega (no perderse si el proceso se cae), se usa una cola real (ver BullMQ) en vez de solo `EventEmitter` en memoria.

## Criterio junior

Un junior debe poder emitir un evento simple y crear un listener con `@OnEvent()` que reaccione a el.

## Criterio mid

Un mid debe decidir que logica pertenece al flujo principal (debe fallar la operacion si falla) y que logica pertenece a un evento desacoplado (no deberia bloquear ni fallar la operacion principal).

## Criterio senior

Un senior debe diseñar el catalogo de eventos de dominio de la aplicacion, decidir cuando `EventEmitter` en memoria alcanza y cuando se necesita una cola persistente, y anticipar que pasa si un listener falla silenciosamente.

## Errores comunes

- Encadenar llamadas directas entre servicios para efectos secundarios en vez de usar eventos, acoplando modulos que no deberian conocerse.
- Usar `EventEmitter` en memoria para efectos que absolutamente deben ejecutarse (por ejemplo, un pago), sin garantia de persistencia si el proceso se cae a mitad de camino.
- No loggear errores dentro de los listeners, dejando fallos silenciosos que nadie nota.
- Nombrar eventos de forma inconsistente entre el equipo, dificultando encontrar quien escucha que.

## Practica

- [ ] Emite un evento de dominio simple desde un servicio existente.
- [ ] Crea un listener que reaccione a ese evento sin acoplarse al servicio emisor.
- [ ] Simula un fallo dentro del listener y verifica que no afecta la operacion principal.
- [ ] Documenta el catalogo de eventos de un proyecto de practica.
- [ ] Anota que efectos de tu dominio deberian ser eventos y cuales deben ser parte del flujo principal.

## Preguntas de repaso

- Que ventaja da desacoplar efectos secundarios con eventos en vez de llamadas directas?
- Por que `EventEmitter` en memoria no es suficiente para efectos que deben garantizar entrega?
- Como deberia nombrarse un evento de dominio y por que?
- Que pasa si un listener falla y no se loggea el error?
