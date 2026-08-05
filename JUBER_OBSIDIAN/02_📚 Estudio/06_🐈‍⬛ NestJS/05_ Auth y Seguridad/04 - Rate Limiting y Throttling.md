---
tags:
  - NestJS
  - TypeScript
  - Nest-Auth
Creado: 2026-07-27
Relacionado:
  - Auth y Seguridad NestJS
  - NESTJS
---

# 04 - Rate Limiting y Throttling

## Que vas a aprender

En este capitulo vas a aprender a proteger una API NestJS contra abuso usando **rate limiting** con `@nestjs/throttler`: limites globales, limites por endpoint, y estrategias para endpoints especialmente sensibles como login.

## Explicacion

Rate limiting restringe cuantos requests puede hacer un mismo cliente (identificado por IP, usuario o API key) en una ventana de tiempo. Sin esto, un cliente malicioso o incluso un bug de un cliente legitimo puede saturar la API, degradando el servicio para todos los demas usuarios.

`@nestjs/throttler` permite configurar limites globales por defecto y sobreescribirlos por controller o endpoint con `@Throttle()`. Los endpoints de autenticacion (login, registro, recuperacion de contraseña) merecen limites mas estrictos que el resto, porque son el blanco tipico de ataques de fuerza bruta.

## Modelo mental

Pensa el rate limiting como un torniquete: no importa cuanta gente quiera entrar, el torniquete solo deja pasar una cantidad controlada por minuto. Esto protege el sistema completo, no solo al usuario abusivo.

## Ejemplo base

```typescript
// app.module.ts
ThrottlerModule.forRoot([{ ttl: 60000, limit: 100 }]); // 100 req/min por default

// Endpoint sensible con limite mas estricto
@Throttle({ default: { limit: 5, ttl: 60000 } }) // 5 intentos por minuto
@Post('login')
login(@Body() dto: LoginDto) {}

// Endpoint publico de alto trafico exento
@SkipThrottle()
@Get('health')
health() {}
```

## Como se usa en un proyecto real

1. Se define un limite global razonable como default para toda la API.
2. Se aplican limites mas estrictos a endpoints de autenticacion y a operaciones costosas (envio de emails, generacion de reportes).
3. Se identifica al cliente por algo mas robusto que solo IP cuando es posible (API key, ID de usuario autenticado), porque muchas IPs son compartidas (NAT, proxies corporativos).
4. Se devuelve el codigo HTTP 429 con headers estandar (`Retry-After`) para que clientes bien comportados puedan reintentar correctamente.
5. Se combina rate limiting a nivel de aplicacion con proteccion a nivel de infraestructura (WAF, API Gateway) cuando el trafico lo justifica.

## Criterio junior

Un junior debe poder configurar un limite global basico con `ThrottlerModule` y aplicar `@Throttle()` a un endpoint especifico.

## Criterio mid

Un mid debe definir limites diferenciados segun la sensibilidad del endpoint, y entender por que limitar solo por IP puede ser insuficiente en ciertos escenarios.

## Criterio senior

Un senior debe diseñar la estrategia de proteccion contra abuso de toda la plataforma (rate limiting de aplicacion + infraestructura, deteccion de patrones anomalos, respuesta ante ataques de fuerza bruta coordinados) y balancear seguridad con experiencia de usuario legitimo.

## Errores comunes

- No aplicar limites especiales a endpoints de login, dejandolos vulnerables a fuerza bruta.
- Limitar solo por IP en un contexto donde muchos usuarios legitimos comparten la misma IP (oficinas, redes moviles con NAT).
- No devolver `Retry-After`, dejando que clientes bien comportados no sepan cuando reintentar.
- Configurar limites tan estrictos que afectan el uso normal de usuarios legitimos.

## Practica

- [ ] Configura un limite global de rate limiting para una API de prueba.
- [ ] Aplica un limite mas estricto a un endpoint de login simulado.
- [ ] Verifica que la respuesta 429 incluya headers utiles para el cliente.
- [ ] Investiga como identificarias clientes de forma mas robusta que solo por IP.
- [ ] Anota que endpoints de tu proyecto merecen limites especiales.

## Preguntas de repaso

- Por que un endpoint de login merece un limite mas estricto que el resto de la API?
- Que problema tiene identificar clientes solo por IP?
- Que informacion deberia incluir una respuesta 429 para ser util al cliente?
- Como combinarias rate limiting de aplicacion con proteccion de infraestructura?
