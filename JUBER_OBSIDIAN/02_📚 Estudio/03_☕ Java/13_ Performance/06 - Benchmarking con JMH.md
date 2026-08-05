---
tags:
  - Java
  - Benchmark
Creado: 2026-05-21
Relacionado:
  - Performance Java
---

# Benchmarking con JMH

JMH es la herramienta correcta para microbenchmarks en Java. Evita errores por JIT, warmup y dead code elimination.

## Ejemplo

```java
@Benchmark
public String concatWithBuilder() {
    StringBuilder builder = new StringBuilder();
    for (int i = 0; i < 100; i++) {
        builder.append(i);
    }
    return builder.toString();
}
```

## Conceptos

| Concepto | Significado |
|---|---|
| Warmup | Iteraciones antes de medir |
| Fork | JVM separada para aislar |
| Throughput | Operaciones por tiempo |
| Average time | Tiempo promedio por operacion |

## Regla

Microbenchmark no reemplaza prueba de carga real. Sirve para comparar piezas pequenas.

## Checklist

- [ ] Uso JMH, no `System.currentTimeMillis`.
- [ ] Configuro warmup.
- [ ] Evito que el compilador elimine el trabajo.
- [ ] Interpreto resultados segun contexto.
