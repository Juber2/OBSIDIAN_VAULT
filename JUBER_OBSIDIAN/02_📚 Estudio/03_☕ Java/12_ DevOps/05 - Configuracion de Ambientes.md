---
tags:
  - Java
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - DevOps con Java
---

# Configuracion de Ambientes

Una aplicacion real cambia configuracion entre local, dev, qa, staging y produccion.

## Orden practico

1. Valores por defecto en `application.yml`.
2. Perfiles por ambiente: `application-dev.yml`, `application-prod.yml`.
3. Variables de entorno para valores desplegables.
4. Secrets externos para credenciales.

## Ejemplo

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
```

## Reglas

- Produccion no debe depender de archivos locales.
- Los secretos no van al repositorio.
- Los defaults deben ser seguros.
- La app debe fallar rapido si falta configuracion critica.

## Checklist

- [ ] Puedo correr local sin tocar produccion.
- [ ] Las credenciales no estan versionadas.
- [ ] Cada ambiente tiene perfil claro.
- [ ] La configuracion esta documentada.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

