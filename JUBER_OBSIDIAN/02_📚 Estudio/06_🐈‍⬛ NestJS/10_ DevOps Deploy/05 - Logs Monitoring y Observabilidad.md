---
tags:
  - NestJS
  - TypeScript
  - Nest-DevOps
Creado: 2026-07-27
Relacionado:
  - DevOps Deploy
  - NESTJS
---

# 05 - Logs Monitoring y Observabilidad

## Que vas a aprender

En este capitulo vas a aprender a instrumentar una aplicacion NestJS para produccion: **logging estructurado**, **health checks**, metricas, y los tres pilares de observabilidad (logs, metricas, traces) aplicados a un backend real.

## Explicacion

El logger por defecto de Nest es util en desarrollo pero limitado en produccion: para poder buscar, filtrar y correlacionar logs a escala se necesita **logging estructurado** (JSON con campos consistentes: timestamp, nivel, request ID, mensaje) integrado con una herramienta centralizada (Datadog, ELK, CloudWatch). El request ID es clave para poder seguir el rastro completo de una request especifica a traves de todos los logs que genero, incluso cruzando llamadas a otros servicios.

`@nestjs/terminus` provee health checks estandarizados (`/health`) que verifican dependencias criticas (base de datos, Redis, servicios externos), usados por el orquestador de infraestructura para saber si una instancia esta lista para recibir trafico o necesita reiniciarse.

## Modelo mental

Pensa la observabilidad como los instrumentos de un avion: los logs son la caja negra (que paso exactamente), las metricas son los indicadores del tablero (velocidad, altitud, en tiempo real), y los traces son el mapa de vuelo completo de una request especifica atravesando todo el sistema. Sin estos tres, volar (operar en produccion) es volar a ciegas.

## Ejemplo base

```typescript
// Logging estructurado con request ID
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const req = context.switchToHttp().getRequest();
    const requestId = req.headers['x-request-id'] ?? randomUUID();
    const logger = new Logger('HTTP');
    return next.handle().pipe(
      tap(() => logger.log(JSON.stringify({ requestId, method: req.method, url: req.url }))),
    );
  }
}

// Health check
@Controller('health')
export class HealthController {
  constructor(private health: HealthCheckService, private db: TypeOrmHealthIndicator) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([() => this.db.pingCheck('database')]);
  }
}
```

## Como se usa en un proyecto real

1. Todo log de produccion se emite en formato estructurado (JSON), nunca texto libre, para que la herramienta de agregacion pueda indexarlo y filtrarlo.
2. Cada request lleva un ID de correlacion que se propaga a traves de todos los logs que genera, incluso al llamar a otros servicios.
3. Se expone un endpoint `/health` que verifica dependencias reales (base de datos, colas, servicios externos criticos), usado por el orquestador para decisiones de routing y reinicio.
4. Se instrumentan metricas clave del negocio y de infraestructura (latencia por endpoint, tasa de error, uso de memoria) con una herramienta de metricas (Prometheus, Datadog).
5. Se configuran alertas basadas en esas metricas, para enterarse de un problema antes de que un usuario lo reporte.

## Criterio junior

Un junior debe poder implementar logging estructurado basico y un endpoint de health check simple.

## Criterio mid

Un mid debe propagar un request ID de correlacion a traves de logs y llamadas a otros servicios, y configurar health checks que verifiquen dependencias reales de la aplicacion.

## Criterio senior

Un senior debe diseñar la estrategia de observabilidad completa del sistema (que se loggea, que metricas importan, que traces se capturan, que alertas existen) y asegurar que un incidente en produccion pueda diagnosticarse en minutos, no en horas.

## Errores comunes

- Loggear en texto libre sin estructura, dificultando busquedas y correlacion a escala.
- No propagar un request ID entre servicios, haciendo casi imposible seguir el rastro de una request que fallo en un sistema distribuido.
- Implementar un health check que siempre responde OK sin verificar dependencias reales, dando falsa sensacion de salud del sistema.
- No configurar alertas, enterandose de problemas de produccion solo cuando un usuario se queja.

## Practica

- [ ] Implementa logging estructurado en JSON para un proyecto Nest de practica.
- [ ] Agrega un request ID que se propague a traves de los logs de una misma request.
- [ ] Implementa un health check que verifique la conexion a base de datos real.
- [ ] Investiga como instrumentar una metrica basica (latencia por endpoint) con Prometheus o equivalente.
- [ ] Anota que alertas configurarias como minimas para tu sistema en produccion.

## Preguntas de repaso

- Por que el logging estructurado es preferible al texto libre en produccion?
- Para que sirve un request ID de correlacion en un sistema con multiples servicios?
- Que deberia verificar un health check para ser realmente util?
- Que diferencia hay entre logs, metricas y traces dentro de observabilidad?
