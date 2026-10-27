---
tags:
  - Java
  - Docker
Creado: 2026-05-21
Relacionado:
  - DevOps con Java
---

# Docker con Java

Docker permite ejecutar tu aplicacion Java en un contenedor reproducible.

## Dockerfile multi-stage

```dockerfile
FROM eclipse-temurin:21-jdk AS build
WORKDIR /app
COPY . .
RUN ./mvnw clean package -DskipTests

FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Buenas practicas

- Usa JRE en runtime si no necesitas JDK.
- No metas secretos dentro de la imagen.
- Usa `.dockerignore`.
- Define memoria segun el contenedor, no segun tu laptop.

## Variables utiles

```bash
docker build -t mi-api:1.0.0 .
docker run -p 8080:8080 -e SPRING_PROFILES_ACTIVE=dev mi-api:1.0.0
```

## Checklist

- [ ] La imagen compila desde cero.
- [ ] El contenedor arranca sin depender del IDE.
- [ ] El puerto esta documentado.
- [ ] No hay credenciales hardcodeadas.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

