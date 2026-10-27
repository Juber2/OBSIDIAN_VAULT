---
tags:
  - Java
  - Docker
Creado: 2026-05-21
Relacionado:
  - DevOps con Java
---

# Docker Compose para Java

Docker Compose sirve para levantar la app con dependencias locales: PostgreSQL, Redis, Kafka, RabbitMQ, etc.

## Ejemplo con PostgreSQL

```yaml
services:
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/app
      SPRING_DATASOURCE_USERNAME: app
      SPRING_DATASOURCE_PASSWORD: app
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
    ports:
      - "5432:5432"
```

## Error comun

`depends_on` no garantiza que PostgreSQL este listo para aceptar conexiones. Para entornos mas serios, usa healthchecks o retry en la app.

## Checklist

- [ ] El equipo puede levantar el entorno con un comando.
- [ ] Las variables estan claras.
- [ ] Los datos locales no se mezclan con produccion.
- [ ] Hay healthcheck si la dependencia tarda en arrancar.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

