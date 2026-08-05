---
tags:
  - PostgreSQL
  - Performance
  - Postgres-Performance-Avanzada
Creado: 2026-07-29
Relacionado:
  - Performance Avanzada
  - POSTGRESQL
---

# 03 - Caching Strategies

## Que vas a aprender

En este capitulo vas a consolidar estrategias de caching especificas para reducir carga sobre Postgres, conectando directamente con el capitulo de Cache del libro de NestJS: cuando cachear a nivel de aplicacion (Redis) versus aprovechar el cache interno de Postgres, y el patron de invalidacion correcta.

## Explicacion

Postgres ya tiene su propio cache interno (`shared_buffers`, ver capitulo de Arquitectura), que mantiene paginas de datos frecuentemente accedidas en memoria automaticamente, sin necesitar configuracion adicional del desarrollador. El **caching a nivel de aplicacion** (Redis, ya visto en el libro de NestJS) es una capa **adicional** por encima de eso, util para evitar no solo el I/O de disco sino tambien el costo de la consulta SQL completa (parsing, planning, ejecucion) para resultados que cambian con poca frecuencia.

La decision de que cachear sigue el mismo principio ya visto en el libro de NestJS: datos de lectura frecuente y baja frecuencia de cambio son buenos candidatos. La diferencia especifica de Postgres es que ciertos resultados (como agregaciones costosas, ver capitulo de Vistas Materializadas de este libro) tienen **dos** opciones de cache: una vista materializada (cache a nivel de base de datos, consultable con SQL normal) o cache en Redis (cache a nivel de aplicacion, mas rapido pero requiere logica de invalidacion en el codigo).

## Modelo mental

Pensa el cache interno de Postgres (`shared_buffers`) como la memoria de corto plazo de alguien muy familiarizado con un archivo fisico: recuerda automaticamente donde estan los documentos que consulto recientemente, sin esfuerzo consciente. El cache de aplicacion (Redis) es como tener una copia fotocopiada de un reporte completo guardada en tu escritorio: evita no solo ir al archivo, sino tambien el trabajo de volver a armar el reporte desde cero cada vez.

## Ejemplo base

```typescript
// Cache en Redis para un resultado agregado costoso (ver capitulo de Cache del libro de NestJS)
@Injectable()
export class ReportesService {
  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,
    @InjectRepository(Pedido) private pedidosRepo: Repository<Pedido>,
  ) {}

  async obtenerTotalVentasMensual(mes: string) {
    const cacheKey = `ventas:${mes}`;
    const cacheado = await this.cache.get(cacheKey);
    if (cacheado) return cacheado;

    const resultado = await this.pedidosRepo
      .createQueryBuilder('p')
      .select('SUM(p.total)', 'total')
      .where("date_trunc('month', p.creadoEn) = :mes", { mes })
      .getRawOne();

    await this.cache.set(cacheKey, resultado, 3600);   // 1 hora
    return resultado;
  }
}
```

```sql
-- Alternativa: vista materializada (ver capitulo de Vistas), consultable con SQL normal
REFRESH MATERIALIZED VIEW resumen_ventas_mensual;
```

## Como se usa en un proyecto real

1. Se confia en `shared_buffers` de Postgres como primera capa de cache automatica, sin necesitar intervencion manual del desarrollador.
2. Se evalua vista materializada (a nivel de base de datos) cuando el resultado costoso se consulta con SQL normal desde multiples lugares, y cache en Redis (a nivel de aplicacion) cuando el resultado se sirve directamente a traves de un endpoint especifico con logica adicional.
3. Se invalida el cache de aplicacion correctamente ante cualquier escritura que afecte el dato cacheado, aplicando el mismo principio ya visto en el capitulo de Cache del libro de NestJS.
4. Se documenta que capa de cache (Postgres interno, vista materializada, Redis) resuelve cada caso especifico, evitando capas redundantes sin beneficio adicional.
5. Se mide el impacto real de cada estrategia de cache antes de asumir su beneficio, aplicando el mismo criterio de evidencia ya visto en el capitulo de Profiling del libro de NestJS.

## Criterio junior

Un junior debe entender que Postgres ya tiene cache interno automatico, y que Redis es una capa adicional para casos especificos.

## Criterio mid

Un mid debe decidir entre vista materializada y cache en Redis para un resultado costoso especifico, segun como se consume ese resultado.

## Criterio senior

Un senior debe diseñar la estrategia de caching completa de un sistema, coordinando las distintas capas (Postgres interno, vistas materializadas, Redis) evitando redundancia innecesaria.

## Errores comunes

- No aprovechar el cache interno de Postgres, intentando resolver con Redis problemas que `shared_buffers` ya resuelve automaticamente.
- Cachear en Redis sin invalidar correctamente ante escrituras, generando el mismo problema de datos viejos ya visto en el libro de NestJS.
- Usar Redis para un resultado que se consulta desde multiples lugares con SQL normal, cuando una vista materializada seria mas simple y consistente.
- No medir el impacto real de una estrategia de cache antes de asumir que aporta beneficio.

## Practica

- [ ] Identifica un resultado costoso de tu proyecto y decide si conviene vista materializada o cache en Redis.
- [ ] Implementa esa estrategia con invalidacion correcta ante escrituras.
- [ ] Mide el impacto real de esa estrategia comparando tiempo de respuesta antes y despues.
- [ ] Documenta que capa de cache resuelve cada caso especifico de tu sistema.
- [ ] Revisa un caso donde Redis se usa innecesariamente para algo que el cache interno de Postgres ya resolvia.

## Preguntas de repaso

- Que hace `shared_buffers` automaticamente sin intervencion del desarrollador?
- Cuando conviene una vista materializada sobre cache en Redis para un resultado costoso?
- Que riesgo comparte cachear en Redis con el mismo problema de invalidacion ya visto en el libro de NestJS?
- Como medirias el impacto real de una estrategia de cache antes de asumir su beneficio?
