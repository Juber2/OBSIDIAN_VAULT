---
tags:
  - Java
  - API
Creado: 2026-05-21
Relacionado:
  - Arquitectura Java
  - Spring Boot
---

# DTOs, Mappers y Contratos

Un DTO representa datos que entran o salen por una frontera: HTTP, cola, archivo o API externa. No es lo mismo que entidad de dominio ni entidad JPA.

## Tipos

| Tipo | Uso |
|---|---|
| Request DTO | Datos que llegan del cliente |
| Response DTO | Datos que devuelves |
| Command | Entrada de un caso de uso |
| Entity JPA | Persistencia |
| Domain model | Reglas de negocio |

## Ejemplo

```java
public record CreateUserRequest(
    String name,
    String email
) {
    CreateUserCommand toCommand() {
        return new CreateUserCommand(name, email);
    }
}
```

## Mapper

```java
class UserResponseMapper {
    UserResponse toResponse(User user) {
        return new UserResponse(
            user.id().value(),
            user.name(),
            user.email().value()
        );
    }
}
```

## MapStruct

MapStruct sirve cuando hay muchos mapeos repetitivos. Para dominios con reglas delicadas, el mapeo manual suele ser mas explicito.

## Checklist

- [ ] No expongo entidades JPA en controllers.
- [ ] No uso el mismo DTO para request y response si tienen reglas distintas.
- [ ] Valido entrada en la frontera.
- [ ] Mantengo versionados los contratos importantes.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

