---
tags:
  - NestJS
  - TypeScript
  - Nest-GraphQL
Creado: 2026-07-27
Relacionado:
  - GraphQL y Frontend
  - NESTJS
---

# 04 - NestJS como Backend para Angular

## Que vas a aprender

En este capitulo vas a aprender los puntos de integracion practicos entre un backend NestJS y un frontend **Angular**: CORS, autenticacion compartida, formatos de error consistentes, y como estructurar contratos que hagan simple el consumo desde `HttpClient`.

## Explicacion

Angular y NestJS comparten filosofia (TypeScript, decoradores, DI, modulos), lo que hace natural mantener consistencia de patrones entre ambos lados del stack: los mismos DTOs (o tipos derivados de ellos) pueden compartirse, los interceptors de Angular (`HttpInterceptor`) reflejan el mismo concepto que los interceptors de Nest, y el manejo de errores puede diseñarse en conjunto para que el frontend sepa exactamente que esperar.

Los puntos de friccion tipicos son: CORS mal configurado bloqueando requests en desarrollo, formatos de error inconsistentes que complican el manejo generico de errores en Angular, y falta de coordinacion en el flujo de autenticacion (donde vive el token, como se refresca, que hace el frontend ante un 401).

## Modelo mental

Pensa el contrato entre NestJS y Angular como un idioma comun: cuanto mas consistente sea el formato de datos, errores y autenticacion del lado del backend, mas simple y generico puede ser el codigo del lado de Angular que consume esa API, sin manejar casos especiales por cada endpoint.

## Ejemplo base

```typescript
// Backend: CORS configurado para el origen exacto de Angular en desarrollo
app.enableCors({ origin: 'http://localhost:4200', credentials: true });

// Angular: interceptor que maneja el refresh de token ante un 401
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((err) => {
      if (err.status === 401) return authService.refresh().pipe(switchMap(() => next(req)));
      return throwError(() => err);
    }),
  );
};
```

## Como se usa en un proyecto real

1. Se define un formato de error unico en el backend (ver Exception Filters) y el frontend construye un manejador generico basado en ese formato, sin parsear casos especiales por endpoint.
2. El flujo de refresh de token (ver capitulo de Auth) se coordina explicitamente: Angular intercepta 401, pide refresh, reintenta la request original.
3. CORS se configura con el origen exacto de cada ambiente de Angular (localhost en dev, dominio real en produccion), nunca con `*` si hay credenciales.
4. Se generan tipos compartidos (ver tema anterior de OpenAPI) para evitar mantener DTOs duplicados a mano en ambos proyectos.
5. Se coordina el versionado de API (ver capitulo de APIs REST) con el ciclo de deploy de Angular, para no romper el frontend en produccion con un cambio de backend.

## Criterio junior

Un junior debe poder configurar CORS basico para que Angular en desarrollo pueda consumir la API sin errores de origen bloqueado.

## Criterio mid

Un mid debe coordinar el flujo completo de autenticacion entre ambos lados (login, refresh automatico ante 401, logout) y mantener el formato de error consistente para simplificar el manejo generico en Angular.

## Criterio senior

Un senior debe definir el proceso de coordinacion de contratos entre los equipos backend y frontend (generacion de tipos, versionado, comunicacion de breaking changes) y asegurar que ambos stacks compartan convenciones que reduzcan friccion de integracion.

## Errores comunes

- Configurar CORS con `origin: '*'` en un proyecto que maneja cookies o tokens, rompiendo la seguridad de esa configuracion.
- Devolver formatos de error distintos en distintos endpoints, obligando a Angular a manejar casos especiales por cada llamada.
- No coordinar el flujo de refresh de token, dejando que el frontend maneje 401 de forma inconsistente en distintos servicios.
- Romper un contrato de API sin avisar al equipo frontend, generando errores en produccion del lado cliente.

## Practica

- [ ] Configura CORS en un backend Nest para un frontend Angular en desarrollo.
- [ ] Implementa un interceptor de Angular que maneje refresh de token ante un 401.
- [ ] Verifica que el formato de error del backend sea consistente en al menos tres endpoints distintos.
- [ ] Genera tipos compartidos a partir de la especificacion OpenAPI del backend (ver tema anterior).
- [ ] Anota que convenciones de tu equipo facilitan (o dificultan) la integracion entre NestJS y Angular.

## Preguntas de repaso

- Por que `origin: '*'` es problematico cuando la API usa cookies o credenciales?
- Como coordinarias el flujo de refresh de token entre backend y frontend?
- Que beneficio da mantener un formato de error consistente en toda la API para el codigo Angular?
- Que pasos seguirias antes de romper un contrato de API que Angular ya consume en produccion?
