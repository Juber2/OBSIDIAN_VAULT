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

# 06 - Microservices y Comunicacion entre Servicios

## Que vas a aprender

En este capitulo vas a aprender a decidir **cuando** migrar de monolito modular a microservicios, y los patrones de resiliencia necesarios (circuit breaker, timeouts, saga) para que un sistema distribuido en NestJS no se vuelva mas fragil que el monolito que reemplaza.

## Explicacion

Microservicios resuelven problemas de escalamiento de equipo (squads independientes, deploys independientes) y de escalamiento tecnico (escalar solo el servicio que lo necesita), pero introducen problemas nuevos que no existen en un monolito: latencia de red, fallos parciales, consistencia eventual entre bases de datos separadas, y complejidad operativa (mas servicios que desplegar, monitorear y versionar).

La regla practica es: no migres a microservicios porque "es lo moderno"; migra cuando el monolito modular realmente choca con un limite (equipos que se pisan constantemente, necesidad real de escalar partes especificas de forma independiente). Cuando se migra, cada llamada entre servicios debe asumir que puede fallar: se necesitan timeouts, reintentos con backoff, circuit breakers, y para operaciones que cruzan varios servicios sin transaccion distribuida, el patron **Saga** (coordinar pasos con compensaciones ante fallo).

## Modelo mental

Pensa la migracion a microservicios como mudarte de una casa (monolito, todo bajo un techo) a un barrio de casas conectadas por calles (servicios conectados por red): ganas espacio independiente por casa, pero ahora dependes de que las calles esten en buen estado, y si una calle se corta, hay que tener un plan B para seguir funcionando.

## Ejemplo base

```typescript
// Circuit breaker simplificado alrededor de una llamada a otro servicio
@Injectable()
export class BillingClient {
  private failures = 0;
  private open = false;

  async charge(orderId: string, amount: number) {
    if (this.open) throw new ServiceUnavailableException('Billing no disponible');
    try {
      const result = await firstValueFrom(
        this.client.send('billing.charge', { orderId, amount }).pipe(timeout(3000)),
      );
      this.failures = 0;
      return result;
    } catch (err) {
      this.failures++;
      if (this.failures >= 5) this.open = true; // abre el circuito tras fallos repetidos
      throw err;
    }
  }
}
```

## Como se usa en un proyecto real

1. Se migra a microservicios de forma incremental, extrayendo primero el modulo mas independiente del monolito modular existente.
2. Toda llamada entre servicios tiene timeout explicito; nunca se espera indefinidamente a otro servicio.
3. Se implementa circuit breaker para evitar que un servicio caido tumbe en cascada a los que dependen de el.
4. Para operaciones multi-servicio sin transaccion distribuida, se implementa el patron Saga con pasos compensables (si el paso 3 falla, se revierten los pasos 1 y 2).
5. Se invierte fuertemente en observabilidad (tracing distribuido, logs correlacionados por request ID) porque debuggear un sistema distribuido sin eso es extremadamente dificil.

## Criterio junior

Un junior debe entender conceptualmente por que una llamada entre microservicios puede fallar de formas que una llamada local no puede, y la importancia de un timeout.

## Criterio mid

Un mid debe implementar manejo de errores y timeouts robustos en llamadas entre servicios, y entender el patron Saga a nivel conceptual para operaciones multi-servicio.

## Criterio senior

Un senior debe decidir cuando migrar de monolito modular a microservicios (o si migrar en absoluto), diseñar la estrategia de resiliencia del sistema distribuido completo, y liderar la inversion en observabilidad necesaria para operar microservicios con confianza.

## Errores comunes

- Migrar a microservicios sin una razon de negocio o de escalamiento de equipo real, pagando la complejidad sin el beneficio.
- No poner timeouts en llamadas entre servicios, permitiendo que un servicio caido cuelgue en cascada a todo el sistema.
- Intentar mantener consistencia transaccional fuerte entre servicios con bases de datos separadas en vez de aceptar consistencia eventual con Saga.
- No invertir en observabilidad distribuida, haciendo que un incidente en produccion sea imposible de diagnosticar rapido.

## Practica

- [ ] Evalua un modulo real de un monolito y decide si justifica migrar a microservicio independiente.
- [ ] Implementa un cliente con timeout explicito para una llamada simulada a otro servicio.
- [ ] Implementa un circuit breaker basico alrededor de esa llamada.
- [ ] Diseña los pasos y compensaciones de una Saga para una operacion multi-servicio de tu dominio.
- [ ] Anota que necesitarias en observabilidad antes de sentirte comodo operando microservicios en produccion.

## Preguntas de repaso

- Que problemas nuevos introducen los microservicios que no existen en un monolito?
- Por que todo timeout en comunicacion entre servicios deberia ser explicito?
- Que resuelve el patron Saga que una transaccion de base de datos tradicional no puede resolver entre servicios distintos?
- Que inversion en observabilidad es indispensable antes de operar microservicios en produccion con confianza?
