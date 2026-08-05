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

# 05 - Helmet CORS y Seguridad HTTP

## Que vas a aprender

En este capitulo vas a aprender a endurecer la capa HTTP de una API NestJS: headers de seguridad con **Helmet**, configuracion correcta de **CORS**, y proteccion basica contra los vectores de ataque web mas comunes.

## Explicacion

**Helmet** configura automaticamente una serie de headers HTTP que mitigan ataques conocidos: `Content-Security-Policy`, `X-Frame-Options` (clickjacking), `Strict-Transport-Security` (fuerza HTTPS), entre otros. Es una linea de defensa barata que deberia activarse en cualquier API expuesta a internet.

**CORS** (Cross-Origin Resource Sharing) controla que origenes (dominios) pueden consumir la API desde un navegador. La configuracion insegura mas comun es `origin: '*'` combinado con `credentials: true`, que en la practica anula la proteccion de CORS para requests autenticadas. La configuracion correcta es una whitelist explicita de origenes permitidos.

## Modelo mental

Pensa Helmet como reforzar las paredes y ventanas del edificio contra ataques conocidos, y CORS como la lista de invitados que el navegador del cliente debe respetar antes de dejar que un sitio web hable con tu API en nombre del usuario logueado.

## Ejemplo base

```typescript
// main.ts
app.use(helmet());

app.enableCors({
  origin: ['https://miapp.com', 'https://admin.miapp.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PATCH', 'DELETE'],
});
```

## Como se usa en un proyecto real

1. Helmet se activa por defecto en cualquier API expuesta, ajustando la Content-Security-Policy segun las necesidades reales del frontend.
2. CORS se configura con una whitelist explicita de dominios permitidos, nunca `origin: '*'` si la API maneja autenticacion con cookies.
3. Se fuerza HTTPS en produccion (redirect automatico o a nivel de infraestructura) y se usa `Strict-Transport-Security`.
4. Se revisan periodicamente las cabeceras de seguridad de la API con herramientas de escaneo (securityheaders.com o equivalente).
5. Se documentan los origenes permitidos y por que, para que agregar uno nuevo sea una decision consciente y revisada.

## Criterio junior

Un junior debe poder activar Helmet y configurar CORS basico con una whitelist de origenes conocidos.

## Criterio mid

Un mid debe entender por que `origin: '*'` con `credentials: true` es peligroso, y ajustar la Content-Security-Policy segun las necesidades reales de la aplicacion sin romper funcionalidad.

## Criterio senior

Un senior debe auditar la postura de seguridad HTTP completa del sistema, coordinar con el equipo de frontend que dominios necesitan acceso, y definir el proceso de revision para agregar nuevos origenes permitidos en CORS.

## Errores comunes

- Configurar `origin: '*'` en una API que maneja cookies de sesion o credenciales.
- No activar Helmet, dejando pasar headers de seguridad basicos que son gratis de implementar.
- Copiar una Content-Security-Policy generica sin ajustarla a las necesidades reales, rompiendo funcionalidad legitima o dejandola demasiado permisiva.
- No revisar periodicamente la configuracion de seguridad HTTP a medida que la aplicacion evoluciona.

## Practica

- [ ] Activa Helmet en un proyecto Nest de prueba.
- [ ] Configura CORS con una whitelist explicita de origenes.
- [ ] Verifica que un origen no autorizado sea rechazado por el navegador.
- [ ] Escanea las cabeceras de seguridad de una API de prueba con una herramienta online.
- [ ] Anota que origenes deberia permitir tu API en produccion y por que.

## Preguntas de repaso

- Que riesgo tiene combinar `origin: '*'` con `credentials: true`?
- Que headers de seguridad configura Helmet por defecto?
- Por que forzar HTTPS es importante incluso si la API "solo" maneja datos no sensibles?
- Como decidirias que dominios agregar a la whitelist de CORS?
