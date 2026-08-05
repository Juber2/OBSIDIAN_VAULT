---
tags:
  - Docker
  - DevOps
  - Docker-Redes
Creado: 2026-07-28
Relacionado:
  - Redes
  - DOCKER
---

# 05 - Comunicacion entre Containers

## Que vas a aprender

En este capitulo vas a consolidar los patrones de comunicacion entre containers de una misma aplicacion: dependencias de arranque (`depends_on` y su limitacion), health checks como señal real de disponibilidad, y patrones de resiliencia basicos para cuando un servicio dependiente no esta listo todavia.

## Explicacion

Un error muy comun es asumir que si el container `db` "arranca antes" que el container `api` (por ejemplo, con `depends_on` en Compose), la base de datos ya esta lista para aceptar conexiones cuando la API arranca. En realidad, `depends_on` sin condicion adicional solo garantiza el **orden de arranque del container**, no que el proceso interno (el motor de base de datos) ya termino su inicializacion y esta aceptando conexiones, lo cual puede tardar varios segundos mas.

La solucion correcta combina **health checks** (que verifican que el servicio realmente responde, no solo que el container esta corriendo) con `depends_on: condition: service_healthy` en Compose, o con logica de reintento con backoff en la propia aplicacion cliente, para tolerar que una dependencia tarde en estar lista sin fallar de forma permanente.

## Modelo mental

Pensa `depends_on` simple como decirle a un mensajero "salí despues de que el otro mensajero haya salido", pero sin garantizar que el otro ya llego a destino. Un health check es la confirmacion real de "ya llegue y estoy listo para recibir", que es la señal que realmente importa antes de intentar comunicarse.

## Ejemplo base

```yaml
services:
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  api:
    build: .
    depends_on:
      db:
        condition: service_healthy  # espera a que el healthcheck de db pase, no solo que arranque
```

```typescript
// Complemento del lado de la aplicacion: reintento con backoff al conectar
async function connectWithRetry(attempts = 5) {
  for (let i = 0; i < attempts; i++) {
    try { return await connect(); }
    catch { await sleep(1000 * (i + 1)); }
  }
  throw new Error('No se pudo conectar tras varios intentos');
}
```

## Como se usa en un proyecto real

1. Se define un health check real (que verifica disponibilidad efectiva, no un simple `exit 0`) para cada servicio con estado del que otros dependen.
2. Se usa `depends_on` con `condition: service_healthy` en Compose, en vez de la forma simple que solo ordena el arranque de containers.
3. Se agrega logica de reintento con backoff en el cliente de la aplicacion como defensa adicional, dado que en produccion (fuera de Compose) esta garantia de orden puede no existir de la misma forma.
4. Se diseñan las aplicaciones para tolerar que una dependencia no este disponible momentaneamente al arrancar, en vez de fallar de forma catastrofica.
5. Se documenta claramente que servicios dependen de cuales, como parte del entendimiento de la arquitectura del sistema.

## Criterio junior

Un junior debe entender que `depends_on` simple no garantiza que un servicio dependiente este realmente listo, solo que arranco antes.

## Criterio mid

Un mid debe configurar health checks reales y usar `condition: service_healthy` para coordinar correctamente el arranque de servicios dependientes.

## Criterio senior

Un senior debe diseñar la resiliencia del sistema completo ante dependencias no disponibles temporalmente (reintentos, circuit breakers, ver tambien el capitulo de microservicios en NestJS), sin depender exclusivamente del orden de arranque de containers.

## Errores comunes

- Confiar solo en `depends_on` simple, asumiendo que garantiza disponibilidad real del servicio dependiente.
- Definir un health check trivial que siempre pasa, perdiendo su valor como señal real de disponibilidad.
- No agregar reintentos del lado de la aplicacion, generando fallos de arranque intermitentes dificiles de reproducir.
- No documentar las dependencias entre servicios, dificultando el diagnostico cuando algo falla en cascada.

## Practica

- [ ] Configura un health check real para un servicio con estado en un `docker-compose.yml`.
- [ ] Usa `depends_on` con `condition: service_healthy` para coordinar el arranque de un servicio dependiente.
- [ ] Implementa logica de reintento con backoff en el cliente de una aplicacion de practica.
- [ ] Simula que la dependencia tarda en estar lista y verifica que el servicio dependiente no falla de forma permanente.
- [ ] Documenta el mapa de dependencias entre servicios de un proyecto real.

## Preguntas de repaso

- Por que `depends_on` simple no garantiza que un servicio dependiente este realmente listo para recibir trafico?
- Que diferencia hay entre un health check trivial y uno que verifica disponibilidad real?
- Que ventaja adicional da agregar reintentos del lado de la aplicacion, ademas de coordinar el orden con Compose?
- Como documentarias las dependencias entre servicios de un sistema con varios containers?
