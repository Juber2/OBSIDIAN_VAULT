---
tags:
  - GitHub
  - Secrets
Creado: 2026-05-22
Relacionado:
  - Seguridad GitHub
---

# Secret Scanning y Prevención

Subir secretos al repo es un incidente de seguridad.

## Prevención

- `.env` en `.gitignore`.
- Usar GitHub Secrets.
- Revisar diff antes de commit.
- Hooks locales si el equipo lo requiere.

## Si subiste un secreto

1. Revoca el secreto.
2. Genera uno nuevo.
3. Evalúa impacto.
4. Limpia historia si es necesario, pero revocar es lo urgente.

## Checklist

- [ ] `.env` ignorado.
- [ ] Secrets no aparecen en commits.
- [ ] Secret scanning habilitado si aplica.
- [ ] Plan de rotación claro.
