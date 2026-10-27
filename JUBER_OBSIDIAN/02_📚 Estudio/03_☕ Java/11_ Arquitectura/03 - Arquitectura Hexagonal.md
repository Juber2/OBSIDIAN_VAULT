---
tags:
  - Java
  - Hexagonal
Creado: 2026-05-21
Relacionado:
  - Arquitectura Java
---

# Arquitectura Hexagonal

Tambien llamada Ports and Adapters. La aplicacion se comunica con el mundo externo mediante puertos. Los adaptadores implementan esos puertos.

## Puertos

| Tipo | Quien lo usa | Ejemplo |
|---|---|---|
| Entrada | El exterior llama a la app | `CreateOrderUseCase` |
| Salida | La app necesita algo externo | `PaymentGateway`, `OrderRepository` |

```java
public interface CreateOrderUseCase {
    OrderId create(CreateOrderCommand command);
}

public interface PaymentGateway {
    PaymentResult charge(Money amount, CardToken token);
}
```

## Adaptadores

```java
@RestController
class OrderController {
    private final CreateOrderUseCase useCase;

    @PostMapping("/orders")
    ResponseEntity<OrderResponse> create(@RequestBody CreateOrderRequest request) {
        OrderId id = useCase.create(request.toCommand());
        return ResponseEntity.created(URI.create("/orders/" + id.value())).build();
    }
}
```

## Beneficio

Puedes cambiar REST por gRPC, JPA por MongoDB o Stripe por otro proveedor sin reescribir el centro de la aplicacion.

## Checklist

- [ ] El dominio no conoce adaptadores.
- [ ] Los adaptadores traducen datos externos.
- [ ] Los puertos tienen nombres de negocio.
- [ ] La infraestructura se puede reemplazar con fakes en tests.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

