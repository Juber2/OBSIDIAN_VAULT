---
tags:
  - NestJS
  - TypeScript
  - Nest-Performance
Creado: 2026-07-27
Relacionado:
  - Performance NestJS
  - NESTJS
---

# 01 - Cache en NestJS

## Que vas a aprender

En este capitulo vas a aprender a usar el modulo de cache de NestJS (`@nestjs/cache-manager`) para reducir carga en base de datos y latencia, con `CacheInterceptor` para cacheo automatico y cache manual para casos mas finos.

## Explicacion

El caching evita recalcular o volver a consultar datos que no cambian con frecuencia. `CacheInterceptor` puede aplicarse a un endpoint entero, cacheando automaticamente la respuesta por un TTL configurado. Para casos mas especificos (cachear solo una parte de un resultado, invalidar cache ante una escritura), se usa `CacheManager` inyectado directamente en el servicio, con control manual de `get`/`set`/`del`.

El desafio real del caching no es agregarlo, es **invalidarlo correctamente**: un cache que devuelve datos viejos despues de una actualizacion es peor que no tener cache, porque genera bugs dificiles de reproducir ("a veces se ve el dato viejo").

## Modelo mental

Pensa el cache como una libreta de notas rapidas al lado del telefono: en vez de llamar de nuevo (consultar la base de datos) cada vez que necesitas un dato que ya preguntaste hace poco, mirás la libreta. El riesgo es no borrar la nota cuando el dato real cambio, y seguir usando informacion vieja sin darte cuenta.

## Ejemplo base

```typescript
@UseInterceptors(CacheInterceptor)
@CacheTTL(60)
@Get('populares')
getPopulares() {
  return this.productosService.findPopulares(); // se cachea automaticamente 60s
}

// Cache manual con invalidacion explicita
@Injectable()
export class ProductosService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  async update(id: string, dto: UpdateProductoDto) {
    const result = await this.productosRepo.update(id, dto);
    await this.cache.del(`producto:${id}`); // invalida al escribir
    return result;
  }
}
```

## Como se usa en un proyecto real

1. Se cachean datos de lectura frecuente y baja frecuencia de cambio (catalogos, configuracion, resultados agregados costosos).
2. Toda operacion de escritura que afecta un dato cacheado invalida explicitamente esa entrada de cache.
3. Se usa Redis como backend de cache (no memoria local del proceso) cuando la aplicacion corre en multiples instancias, para que todas compartan el mismo estado de cache.
4. Se define un TTL conservador segun que tan critico es que el dato este siempre actualizado.
5. Se documenta que datos estan cacheados y con que estrategia de invalidacion, para que el equipo no se sorprenda con datos "desactualizados" en debugging.

## Criterio junior

Un junior debe poder aplicar `CacheInterceptor` a un endpoint de lectura simple con un TTL basico.

## Criterio mid

Un mid debe implementar invalidacion manual de cache ante operaciones de escritura, y elegir Redis como backend cuando la aplicacion corre en multiples instancias.

## Criterio senior

Un senior debe diseñar la estrategia de caching de todo el sistema (que se cachea, con que TTL, como se invalida, que pasa ante un cache stampede) y balancear consistencia de datos con performance segun la criticidad de cada dato.

## Errores comunes

- Cachear datos sin invalidar al escribir, generando bugs de "datos viejos" dificiles de reproducir.
- Usar cache en memoria local del proceso en una aplicacion con multiples instancias, generando inconsistencia entre lo que ve cada instancia.
- Cachear datos altamente sensibles al tiempo real sin evaluar el impacto de mostrar informacion desactualizada.
- No tener visibilidad de que esta cacheado, complicando el debugging cuando algo "no se actualiza".

## Practica

- [ ] Aplica `CacheInterceptor` a un endpoint de lectura de un proyecto de practica.
- [ ] Implementa invalidacion manual de esa cache al ejecutar la operacion de escritura correspondiente.
- [ ] Configura Redis como backend de cache en vez de memoria local.
- [ ] Simula el problema de datos viejos por no invalidar cache y luego corrigelo.
- [ ] Anota que datos de tu sistema son buenos candidatos a cache y cuales no.

## Preguntas de repaso

- Por que invalidar el cache correctamente es mas dificil (y mas importante) que simplemente agregarlo?
- Por que el cache en memoria local del proceso es problematico con multiples instancias?
- Que criterio usarias para decidir el TTL de un dato cacheado?
- Que riesgo tiene cachear un dato altamente sensible al tiempo real?
