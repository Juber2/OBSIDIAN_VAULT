---
tags:
  - NET
  - OWASP
Creado: 2026-05-21
Relacionado:
  - Seguridad en .NET
---

# OWASP para APIs .NET

OWASP resume riesgos comunes de seguridad en aplicaciones y APIs.

## Riesgos frecuentes

| Riesgo | Prevención |
|---|---|
| Broken access control | permisos por recurso |
| Injection | queries parametrizadas |
| Sensitive data exposure | no loguear secretos |
| Security misconfiguration | defaults seguros |
| Vulnerable dependencies | actualizar paquetes |

## Checklist

- [ ] Valido permisos en backend.
- [ ] Uso parámetros en SQL.
- [ ] No expongo stack traces.
- [ ] Reviso dependencias vulnerables.
- [ ] Tengo rate limiting si aplica.
