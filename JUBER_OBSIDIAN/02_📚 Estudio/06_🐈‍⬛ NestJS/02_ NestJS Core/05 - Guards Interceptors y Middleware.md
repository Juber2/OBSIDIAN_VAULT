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

# 05 - Guards Interceptors y Middleware

## Que vas a aprender

En este capitulo vas a diferenciar los tres mecanismos que Nest ofrece para interceptar el ciclo de un request: **middleware**, **guards** e **interceptors**, y saber exactamente cuando usar cada uno.

## Explicacion

**Middleware** corre primero, antes que nada dentro de Nest (equivalente a middleware de Express); es lo mas cercano a la capa HTTP cruda (logging de requests, parseo de body). **Guards** deciden si un request puede continuar o no, principalmente para autenticacion y autorizacion (`canActivate`); si un guard devuelve `false`, Nest corta con 403 automaticamente. **Interceptors** envuelven la ejecucion del handler: pueden transformar la respuesta, medir tiempos, agregar logging, cachear resultados o manejar errores de forma transversal, corriendo antes y despues del handler.

El orden de ejecucion es: Middleware → Guards → Interceptors (parte "antes") → Pipes → Handler → Interceptors (parte "despues") → Exception Filters (si algo falla).

## Modelo mental

Pensa el request como un pasillo con puertas: el middleware es la puerta de entrada al edificio, los guards son el control de acceso a una sala especifica ("¿podes entrar?"), y los interceptors son camaras que registran todo lo que pasa antes y despues de que entres, y pueden incluso modificar lo que sale.

## Ejemplo base

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return Boolean(request.headers.authorization);
  }
}

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const start = Date.now();
    return next.handle().pipe(
      tap(() => console.log(`Request tomo ${Date.now() - start}ms`)),
    );
  }
}

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.url}`);
    next();
  }
}
```

## Como se usa en un proyecto real

1. Middleware para necesidades de bajo nivel: logging crudo, parseo de headers, CORS manual.
2. Guards para toda decision de "puede o no puede" acceder (autenticacion, roles, permisos, feature flags).
3. Interceptors para transformar respuestas (envelope estandar), medir performance, cachear, o mapear errores de infraestructura a excepciones de dominio.
4. Se aplican de forma global cuando el criterio es transversal a toda la app, y por controller/ruta cuando es especifico.
5. Se testean guards e interceptors de forma unitaria, mockeando el `ExecutionContext`.

## Criterio junior

Un junior debe poder diferenciar conceptualmente middleware, guards e interceptors, y aplicar un guard de autenticacion basico a una ruta.

## Criterio mid

Un mid debe crear guards de autorizacion basados en roles y interceptors para necesidades transversales (logging, formato de respuesta), aplicandolos en el nivel correcto (global, controller o metodo).

## Criterio senior

Un senior debe diseñar la estrategia completa de cross-cutting concerns de la aplicacion (que va en middleware, que en guards, que en interceptors, que en exception filters) y evitar que esa logica transversal termine duplicada dentro de los servicios de negocio.

## Errores comunes

- Poner logica de autorizacion dentro del controller o servicio en vez de un guard.
- Usar middleware para logica que necesita el contexto de ejecucion de Nest (guards/interceptors tienen acceso a metadata de decoradores, middleware no).
- No testear guards e interceptors por considerarlos "codigo de infraestructura" de bajo riesgo.
- Duplicar la misma logica transversal (logging, manejo de errores) en cada controller en vez de centralizarla.

## Practica

- [ ] Crea un guard de autenticacion basico y aplicalo a un controller.
- [ ] Crea un interceptor que mida el tiempo de respuesta de cada request.
- [ ] Crea un middleware simple de logging y compara su alcance con el de un interceptor.
- [ ] Escribe un test unitario para el guard, mockeando el `ExecutionContext`.
- [ ] Anota una tabla propia: que problema resuelve cada uno de los tres mecanismos.

## Preguntas de repaso

- En que orden se ejecutan middleware, guards e interceptors?
- Por que un guard no deberia usarse para transformar la respuesta?
- Que puede hacer un interceptor que un guard no puede?
- Que ventaja tiene un guard de roles centralizado sobre validar el rol dentro de cada servicio?
