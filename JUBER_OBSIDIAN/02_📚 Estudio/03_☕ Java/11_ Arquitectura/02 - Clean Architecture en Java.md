---
tags:
  - Java
  - Clean Architecture
Creado: 2026-05-21
Relacionado:
  - Arquitectura Java
---

# Clean Architecture en Java

Clean Architecture organiza el codigo para que las reglas de negocio sean independientes de frameworks, base de datos y UI.

## Dependencias

La direccion de dependencia va hacia adentro:

```text
controllers -> usecases -> domain
repositories jpa -> ports -> domain/usecases
```

## Estructura posible

```text
src/main/java/com/app
  domain/
    model/
    policy/
  application/
    port/in/
    port/out/
    usecase/
  infrastructure/
    persistence/
    web/
    security/
```

## Puerto de salida

```java
public interface UserRepository {
    Optional<User> findByEmail(Email email);
    void save(User user);
}
```

## Adaptador JPA

```java
@Repository
class JpaUserRepositoryAdapter implements UserRepository {
    private final SpringDataUserRepository repository;
    private final UserMapper mapper;

    public void save(User user) {
        repository.save(mapper.toEntity(user));
    }
}
```

## Cuando usarla

Usala cuando hay reglas de negocio importantes, integraciones externas o vida larga del producto. Para CRUD simple puede ser demasiado ceremonial.

## Checklist

- [ ] Las entidades de dominio no tienen anotaciones JPA.
- [ ] Los casos de uso son faciles de testear sin Spring.
- [ ] Los adaptadores convierten entre modelos externos y modelo interno.
- [ ] Los nombres reflejan intencion de negocio, no solo tecnologia.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

