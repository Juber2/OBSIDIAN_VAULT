---
tags:
  - Java
  - Cache
Creado: 2026-05-21
Relacionado:
  - Performance Java
---

# Cache con Redis y Caffeine

Cache guarda resultados para evitar recalcular o consultar datos repetidos.

## Tipos

| Cache | Donde vive | Uso |
|---|---|---|
| Caffeine | Memoria de la app | Rapida, local |
| Redis | Servicio externo | Compartida entre instancias |

## Spring Cache

```java
@Cacheable(value = "users", key = "#id")
public UserResponse findById(Long id) {
    return repository.findById(id)
        .map(mapper::toResponse)
        .orElseThrow();
}
```

## Invalidacion

```java
@CacheEvict(value = "users", key = "#id")
public void updateUser(Long id, UpdateUserRequest request) {
    // update
}
```

## Riesgos

- Datos stale.
- Cache stampede.
- Memoria sin limite.
- Guardar datos sensibles.

## Checklist

- [ ] Defino TTL.
- [ ] Defino estrategia de invalidacion.
- [ ] Mido hit ratio.
- [ ] No cacheo errores permanentes sin pensar.
