---
tags:
  - Git
  - Configuracion
Creado: 2026-05-22
Relacionado:
  - Git Fundamentos
---

# Configuración Inicial Git

Antes de trabajar, configura identidad y preferencias básicas.

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"
```

## Ver configuración

```bash
git config --list
git config --global --list
```

## Checklist

- [ ] Nombre configurado.
- [ ] Email configurado.
- [ ] Rama default en `main`.
- [ ] Editor configurado.
