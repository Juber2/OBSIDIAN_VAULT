---
tags:
  - Java
  - Spring
  - Security
Creado: 2026-05-21
Relacionado:
  - Spring Boot
  - JWT
  - OAuth2
Palabras clave:
  - Spring Security, autenticación, autorización, JWT, OAuth2, CORS, CSRF
---

#Principal

# 🔐 Spring Security

## Progreso

| Tema | Estado |
|------|--------|
| [[01 - Fundamentos Spring Security]] | ⬜ |
| [[02 - JWT]] | ⬜ |
| [[03 - OAuth2 y OpenID Connect]] | ⬜ |
| [[04 - Roles y Permisos]] | ⬜ |
| [[05 - CORS y CSRF]] | ⬜ |

## 🧠 ¿Qué es Spring Security?

Framework de seguridad para aplicaciones Spring. Maneja:
- **Autenticación**: quién eres (login)
- **Autorización**: qué puedes hacer (permisos)

## Flujo de una request

```
HTTP Request
    ↓
SecurityFilterChain (cadena de filtros)
    ↓
AuthenticationFilter (¿tiene credenciales?)
    ↓
AuthenticationManager
    ↓
UserDetailsService (¿existe el usuario?)
    ↓
PasswordEncoder (¿contraseña válida?)
    ↓
SecurityContext (guarda la autenticación)
    ↓
Authorization (¿tiene permisos para este endpoint?)
    ↓
Controller / 403 Forbidden
```

## Conceptos clave

| Concepto | Qué es |
|----------|--------|
| `SecurityFilterChain` | Pipeline de filtros que procesa cada request |
| `UserDetailsService` | Carga el usuario desde la fuente de datos |
| `PasswordEncoder` | Hashea y verifica contraseñas (BCrypt) |
| `Authentication` | Representa al usuario autenticado |
| `SecurityContext` | Holder de la autenticación actual |
| `GrantedAuthority` | Un permiso o rol del usuario |

## Dependencia

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

> [!warning] Solo agregar la dependencia activa protección total
> Todos los endpoints quedan protegidos automáticamente. Necesitas configurar para abrir los que quieras públicos.

---

## ✍️ Mis apuntes
> 📓 Apuntes del módulo → [[Apuntes - Spring]]

<!-- Anotaciones rápidas sobre el módulo en general -->

