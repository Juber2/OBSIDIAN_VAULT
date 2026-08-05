---
tags:
  - Java
  - Arquitectura
Creado: 2026-05-21
Relacionado:
  - Arquitectura Java
---

# Capas y Responsabilidades

Una aplicacion Java mantenible suele separar el codigo por responsabilidades. La idea no es tener muchas carpetas, sino evitar que una clase haga de todo.

## Capas comunes

| Capa | Responsabilidad | Ejemplo |
|---|---|---|
| Controller | HTTP, parametros, status codes | `UserController` |
| Application/Use case | Orquestar reglas del caso de uso | `CreateUserUseCase` |
| Domain | Reglas del negocio | `User`, `Email`, `Order` |
| Infrastructure | Base de datos, APIs externas, archivos | `JpaUserRepository` |

## Regla senior

Las capas internas no deberian depender de detalles externos. El dominio no deberia saber que existe Spring, JPA, Kafka o HTTP.

```java
public class RegisterUserUseCase {
    private final UserRepository repository;

    public RegisterUserUseCase(UserRepository repository) {
        this.repository = repository;
    }

    public UserId execute(RegisterUserCommand command) {
        User user = User.register(command.email(), command.name());
        repository.save(user);
        return user.id();
    }
}
```

## Error comun

```java
@RestController
class UserController {
    @PostMapping("/users")
    void create(@RequestBody UserEntity entity) {
        repository.save(entity);
    }
}
```

El controller esta recibiendo entidad de base de datos, decide el caso de uso y persiste directamente. Funciona, pero escala mal.

## Checklist

- [ ] Controller no contiene reglas de negocio.
- [ ] Use case no depende de HTTP.
- [ ] Dominio no depende de Spring.
- [ ] Infraestructura implementa contratos, no define el negocio.
