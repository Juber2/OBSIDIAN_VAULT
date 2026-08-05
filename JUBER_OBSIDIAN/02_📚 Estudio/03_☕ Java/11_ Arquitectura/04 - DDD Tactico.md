---
tags:
  - Java
  - DDD
Creado: 2026-05-21
Relacionado:
  - Arquitectura Java
---

# DDD Tactico

Domain-Driven Design ayuda a modelar software alrededor del negocio. La parte tactica se enfoca en patrones de codigo.

## Building blocks

| Concepto | Significado | Ejemplo |
|---|---|---|
| Entity | Tiene identidad | `User`, `Order` |
| Value Object | Se compara por valor | `Email`, `Money` |
| Aggregate | Grupo consistente | `Order` con `OrderLine` |
| Repository | Coleccion de agregados | `OrderRepository` |
| Domain Service | Regla que no pertenece a una entidad | `PricingPolicy` |

## Value Object

```java
public record Email(String value) {
    public Email {
        if (value == null || !value.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
    }
}
```

## Aggregate

```java
public class Order {
    private final OrderId id;
    private final List<OrderLine> lines = new ArrayList<>();

    public void addProduct(Product product, int quantity) {
        if (quantity <= 0) throw new IllegalArgumentException("Quantity must be positive");
        lines.add(new OrderLine(product.id(), quantity, product.price()));
    }
}
```

## Error comun

Usar DDD para renombrar carpetas sin modelar reglas reales. DDD se justifica cuando el dominio tiene complejidad.

## Checklist

- [ ] Uso value objects para valores con reglas.
- [ ] Protejo invariantes dentro del agregado.
- [ ] No expongo listas mutables del dominio.
- [ ] El lenguaje del codigo se parece al lenguaje del negocio.
