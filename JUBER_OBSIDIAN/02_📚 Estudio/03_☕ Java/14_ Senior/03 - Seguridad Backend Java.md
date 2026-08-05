---
tags:
  - Java
  - Seguridad
Creado: 2026-05-21
Relacionado:
  - Senior Java Engineer
  - Spring Security
---

# Seguridad Backend Java

Seguridad no es solo login. Incluye autorizacion, validacion, secretos, auditoria y manejo seguro de errores.

## Riesgos comunes

| Riesgo | Prevencion |
|---|---|
| Broken access control | Validar permisos por recurso |
| Injection | Prepared statements, validacion |
| Secret leakage | Secrets manager, no logs sensibles |
| JWT debil | Expiracion, firma fuerte, audience |
| Error leakage | Respuestas genericas, logs internos |

## Autorizacion por recurso

No basta con `ROLE_ADMIN` o `ROLE_USER`. Pregunta si el usuario puede acceder a ese recurso especifico.

```java
if (!currentUser.canAccess(order)) {
    throw new AccessDeniedException("Forbidden");
}
```

## Checklist

- [ ] Valido autenticacion y autorizacion.
- [ ] No confio en ids enviados por cliente.
- [ ] No logueo tokens ni passwords.
- [ ] Manejo errores sin revelar detalles internos.
- [ ] Mantengo dependencias actualizadas.
