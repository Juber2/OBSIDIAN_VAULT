---
tags:
  - NestJS
  - TypeScript
  - Nest-APIs
Creado: 2026-07-27
Relacionado:
  - APIs REST NestJS
  - NESTJS
---

# 03 - Exception Filters y Manejo de Errores

## Que vas a aprender

En este capitulo vas a aprender a centralizar el manejo de errores en NestJS con **exception filters**, la jerarquia de `HttpException`, y como devolver respuestas de error consistentes en toda la API.

## Explicacion

Nest ya maneja excepciones HTTP estandar (`NotFoundException`, `BadRequestException`, `ForbiddenException`, etc.) convirtiendolas automaticamente en la respuesta HTTP correspondiente. El problema aparece con errores que **no** son `HttpException`: errores de base de datos, errores de librerias externas, bugs inesperados. Sin un filtro global, esos errores pueden filtrar detalles internos (stack traces, mensajes de driver de base de datos) al cliente.

Un **exception filter global** (`@Catch()`) centraliza esa traduccion: captura cualquier error no manejado, lo loggea con detalle en el servidor, y devuelve al cliente una respuesta generica y segura, con un formato consistente en toda la API.

## Modelo mental

Pensa el exception filter global como una red de seguridad debajo de toda la aplicacion: pase lo que pase arriba (un bug, un timeout de base de datos, una libreria que tira un error raro), nada llega al cliente sin pasar antes por esa red, que decide que informacion es segura de mostrar.

## Ejemplo base

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    const message = exception instanceof HttpException
      ? exception.getResponse()
      : 'Error interno del servidor';

    if (status === 500) console.error(exception); // logging detallado interno

    response.status(status).json({
      statusCode: status,
      message,
      timestamp: new Date().toISOString(),
    });
  }
}

app.useGlobalFilters(new AllExceptionsFilter());
```

## Como se usa en un proyecto real

1. Se registra un exception filter global que captura absolutamente todo, incluso errores no previstos.
2. Se define un formato de respuesta de error estandar y documentado para toda la API.
3. Los errores 500 nunca exponen detalles internos al cliente, pero se loggean completos en el servidor (idealmente con correlacion a un request ID).
4. Se crean excepciones de dominio propias cuando `HttpException` no expresa bien el error de negocio, y se mapean a HTTP en el filtro.
5. Se integra el logging de errores con la herramienta de observabilidad del equipo (Sentry, Datadog, etc.).

## Criterio junior

Un junior debe entender la jerarquia basica de `HttpException` y usar las excepciones predefinidas de Nest en servicios y controllers.

## Criterio mid

Un mid debe crear un exception filter global que capture errores no manejados y devuelva un formato de error consistente sin filtrar detalles internos.

## Criterio senior

Un senior debe diseñar la taxonomia completa de errores del sistema (errores de validacion, de negocio, de infraestructura), integrar el manejo de errores con observabilidad, y asegurar trazabilidad (request ID) entre lo que ve el cliente y lo que se loggea internamente.

## Errores comunes

- No tener un exception filter global y dejar que errores no manejados devuelvan stack traces al cliente.
- Devolver formatos de error distintos en cada endpoint, dificultando el consumo desde el frontend.
- Loggear errores sin suficiente contexto (usuario, request, payload) para poder diagnosticarlos despues.
- Capturar excepciones silenciosamente sin loggearlas, perdiendo visibilidad de fallos reales en produccion.

## Practica

- [ ] Crea un exception filter global que capture cualquier error no manejado.
- [ ] Define un formato de respuesta de error estandar para tu API.
- [ ] Crea una excepcion de dominio propia y mapeala a un codigo HTTP en el filtro.
- [ ] Verifica que un error 500 nunca exponga detalles internos al cliente.
- [ ] Integra el logging de errores con una herramienta de observabilidad (aunque sea simulada).

## Preguntas de repaso

- Que riesgo tiene no tener un exception filter global?
- Como asegurarias que un error 500 no exponga detalles internos al cliente?
- Cuando crearias una excepcion de dominio propia en vez de usar las de Nest directamente?
- Que informacion deberia guardar un log de error para que sea util al diagnosticar un incidente?
