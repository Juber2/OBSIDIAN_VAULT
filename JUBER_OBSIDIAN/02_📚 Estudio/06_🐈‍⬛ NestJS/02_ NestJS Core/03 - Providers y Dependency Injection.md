---
tags:
  - NestJS
  - TypeScript
  - Nest-Core
Creado: 2026-07-27
Relacionado:
  - NestJS Core
  - NESTJS
---

# 03 - Providers y Dependency Injection

## Que vas a aprender

En este capitulo vas a entender el corazon de NestJS: la **inyeccion de dependencias (DI)**. Vas a ver que es un provider, los distintos scopes (singleton, request, transient), y como inyectar valores, tokens custom e interfaces.

## Explicacion

Un provider es cualquier clase decorada con `@Injectable()` que Nest puede instanciar y entregar a quien la necesite (constructor injection). Por defecto, los providers son **singleton**: una unica instancia compartida por toda la aplicacion. Nest tambien soporta scope `REQUEST` (una instancia por request HTTP, util para contexto de usuario autenticado) y `TRANSIENT` (una instancia nueva cada vez que se inyecta).

Como TypeScript no tiene interfaces en runtime, cuando se quiere inyectar algo que no es una clase concreta (una config, una implementacion intercambiable) se usa un **token de inyeccion** (string o `Symbol`) junto con `@Inject()`, y se lo registra explicitamente en el modulo.

## Modelo mental

Pensa el contenedor de DI de Nest como una caja de herramientas que arma automaticamente cada objeto que necesitas, resolviendo primero todas sus dependencias. Vos declaras "que necesito" en el constructor, y Nest se encarga de "como se construye".

## Ejemplo base

```typescript
export const MAIL_SENDER = Symbol('MAIL_SENDER');

export interface MailSender {
  send(to: string, subject: string): Promise<void>;
}

@Injectable()
export class SmtpMailSender implements MailSender {
  async send(to: string, subject: string) {
    // implementacion real
  }
}

@Module({
  providers: [{ provide: MAIL_SENDER, useClass: SmtpMailSender }],
  exports: [MAIL_SENDER],
})
export class MailModule {}

@Injectable()
export class UsersService {
  constructor(@Inject(MAIL_SENDER) private readonly mailer: MailSender) {}
}
```

## Como se usa en un proyecto real

1. Se usan clases concretas como providers para el caso comun (servicios, repositorios).
2. Se usan tokens custom cuando se necesita desacoplar una interfaz de su implementacion (por ejemplo, para poder mockearla en tests o cambiarla por entorno).
3. El scope `REQUEST` se reserva para casos donde realmente se necesita estado por request (tenant actual, usuario autenticado); tiene costo de performance porque no se puede cachear como singleton.
4. Se evita inyectar el modulo de configuracion completo cuando solo se necesita un valor puntual; se expone a traves de un provider dedicado.
5. Se documenta que providers son intercambiables por entorno (test, dev, prod) usando `useValue`, `useFactory` o `useClass`.

## Criterio junior

Un junior debe entender que es `@Injectable()`, como se inyecta por constructor, y usar el scope default (singleton) correctamente.

## Criterio mid

Un mid debe saber cuando usar un token custom con `@Inject()`, cuando usar `useFactory` para providers que dependen de configuracion async, y entender el costo del scope `REQUEST`.

## Criterio senior

Un senior debe diseñar abstracciones (interfaces + tokens) para las dependencias que probablemente cambien entre entornos o proveedores externos, y decidir cuando vale la pena ese desacople extra frente a la simplicidad de una clase concreta.

## Errores comunes

- Usar el scope `REQUEST` en todo "por si acaso", degradando performance sin necesidad.
- No usar tokens custom y terminar acoplado a una implementacion concreta imposible de mockear en tests.
- Instanciar dependencias manualmente con `new` dentro de un provider en vez de dejar que Nest las inyecte.
- Registrar el mismo provider en multiples modulos sin exportarlo desde un lugar central, duplicando instancias sin darse cuenta.

## Practica

- [ ] Crea un provider con una interfaz y un token custom.
- [ ] Cambia la implementacion inyectada usando `useClass` segun un valor de configuracion.
- [ ] Crea un provider con `useFactory` que dependa de otro provider async.
- [ ] Experimenta con scope `REQUEST` y mide el impacto conceptual en memoria/performance.
- [ ] Anota cuando en tu proyecto usarias token custom vs clase concreta directa.

## Preguntas de repaso

- Que diferencia hay entre `useClass`, `useValue` y `useFactory`?
- Por que necesitas un token de inyeccion para una interfaz?
- Que costo tiene el scope `REQUEST` frente al singleton default?
- Como mockearias una dependencia en un test si usa un token custom?
