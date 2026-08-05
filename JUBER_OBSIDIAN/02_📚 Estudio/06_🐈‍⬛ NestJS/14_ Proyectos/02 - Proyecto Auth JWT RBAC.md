---
tags:
  - NestJS
  - TypeScript
  - Nest-Proyectos
Creado: 2026-07-27
Relacionado:
  - Proyectos
  - NESTJS
---

# 02 - Proyecto Auth JWT RBAC

## Que vas a construir

Un sistema de autenticacion y autorizacion completo sobre el proyecto CRUD anterior (o uno nuevo): login con JWT (access + refresh), roles, y al menos una regla de autorizacion condicional (ownership) con CASL.

## Alcance funcional

- Registro y login con `LocalStrategy` + `JwtStrategy`.
- Access token de corta duracion y refresh token con rotacion, registrado en base de datos para poder revocarse.
- Endpoint de logout que revoca el refresh token.
- RBAC simple (`@Roles()` + `RolesGuard`) para al menos un endpoint administrativo.
- Una regla de autorizacion condicional con CASL (por ejemplo, "un usuario solo puede editar sus propios recursos").
- Rate limiting estricto en el endpoint de login.

## Checklist tecnica

- [ ] Los mensajes de error de login no revelan si el email existe o no.
- [ ] El refresh token se guarda en almacenamiento seguro del lado cliente (no `localStorage` si hay alternativa).
- [ ] `ClassSerializerInterceptor` activo, ningun campo sensible (password, tokens) sale en ninguna respuesta.
- [ ] Guards de autenticacion y autorizacion aplicados en el orden correcto, con tests unitarios propios.
- [ ] Helmet y CORS configurados correctamente para el frontend que consumira la API.
- [ ] Tests que verifiquen: un usuario no puede editar el recurso de otro; un rol sin permiso recibe 403.

## Criterio de exito

El proyecto se considera completo cuando existe un flujo verificable de: registro, login, acceso a un recurso protegido, refresh de token expirado, intento fallido de acceder a un recurso ajeno (rechazado correctamente), y logout que efectivamente invalida el refresh token.

## Errores comunes a evitar

- Un unico token de larga duracion sin mecanismo de revocacion.
- No testear el caso de autorizacion condicional (ownership), solo el RBAC simple.
- Rate limiting generico sin un limite mas estricto especifico para el login.
- CORS con `origin: '*'` mientras se usan credenciales.

## Extension opcional

Agregar login social (OAuth2 con Google o GitHub) manejando correctamente el caso de un email que ya existe con login tradicional.
