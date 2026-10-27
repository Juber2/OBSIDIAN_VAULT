---
tags:
  - Java
  - JVM
Creado: 2026-05-21
Relacionado:
  - Performance Java
---

# Memory Leaks

En Java hay memory leak cuando objetos que ya no deberian usarse siguen referenciados.

## Causas comunes

- Caches sin limite.
- Listas o mapas estaticos.
- Listeners no removidos.
- ThreadLocal sin limpiar.
- Sesiones con demasiados datos.

## Ejemplo

```java
class BadCache {
    private static final Map<String, byte[]> cache = new HashMap<>();

    void put(String key, byte[] value) {
        cache.put(key, value);
    }
}
```

## Correccion

Usa caches con expiracion y limite:

```java
Cache<String, User> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(Duration.ofMinutes(10))
    .build();
```

## Checklist

- [ ] Reviso heap dump.
- [ ] Identifico dominator objects.
- [ ] Busco colecciones que crecen sin limite.
- [ ] Limpio ThreadLocal en finally si aplica.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

