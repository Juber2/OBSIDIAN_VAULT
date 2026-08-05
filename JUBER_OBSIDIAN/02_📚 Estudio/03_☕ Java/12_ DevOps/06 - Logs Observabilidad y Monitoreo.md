---
tags:
  - Java
  - Observabilidad
Creado: 2026-05-21
Relacionado:
  - DevOps con Java
---

# Logs, Observabilidad y Monitoreo

Un senior no solo escribe codigo: tambien deja la aplicacion observable para diagnosticar problemas.

## Tres pilares

| Pilar | Pregunta que responde |
|---|---|
| Logs | Que paso |
| Metricas | Cuanto y con que tendencia |
| Traces | Donde se fue el tiempo entre servicios |

## Logging practico

```java
private static final Logger log = LoggerFactory.getLogger(UserService.class);

public User create(CreateUserCommand command) {
    log.info("Creating user email={}", command.email());
    return repository.save(User.create(command.email()));
}
```

No loguees passwords, tokens, tarjetas ni datos personales innecesarios.

## Spring Boot Actuator

Actuator expone health, metrics, info y otros endpoints utiles para operacion.

## Checklist

- [ ] Los logs tienen contexto suficiente.
- [ ] No hay informacion sensible en logs.
- [ ] Hay health checks.
- [ ] Hay metricas de negocio y tecnicas.
- [ ] Puedo correlacionar una request entre servicios.
