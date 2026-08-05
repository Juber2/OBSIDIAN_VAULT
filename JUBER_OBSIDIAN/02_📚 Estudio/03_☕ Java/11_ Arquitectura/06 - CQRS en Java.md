---
tags:
  - Java
  - CQRS
Creado: 2026-05-21
Relacionado:
  - Arquitectura Java
---

# CQRS en Java

CQRS separa operaciones de escritura (commands) y lectura (queries). No obliga a usar dos bases de datos.

## Idea

| Lado | Intencion | Retorno |
|---|---|---|
| Command | Cambiar estado | id, void o resultado minimo |
| Query | Consultar estado | DTO optimizado para lectura |

```java
public record CreateProductCommand(String name, BigDecimal price) {}

public interface CreateProductUseCase {
    ProductId execute(CreateProductCommand command);
}
```

```java
public interface ProductQueryService {
    List<ProductListItem> findAvailableProducts();
}
```

## Ventaja

Evita mezclar reglas de escritura con pantallas de lectura. Puedes optimizar consultas sin contaminar el modelo de dominio.

## Cuando evitarlo

Si tu sistema es CRUD pequeno, CQRS completo puede meter complejidad innecesaria. Puedes aplicar separacion ligera: services de comandos y services de queries.

## Checklist

- [ ] Los commands expresan intencion, no setters tecnicos.
- [ ] Las queries devuelven DTOs de lectura.
- [ ] Las escrituras validan invariantes.
- [ ] No uso CQRS como excusa para duplicar todo sin necesidad.
