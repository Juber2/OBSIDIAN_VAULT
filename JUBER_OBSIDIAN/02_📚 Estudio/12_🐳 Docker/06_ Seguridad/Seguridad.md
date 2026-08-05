---
tags:
  - Docker
  - DevOps
  - Docker-Seguridad
Creado: 2026-07-28
Relacionado:
  - DOCKER
---

# Seguridad

## Proposito del capitulo

Este capitulo cubre seguridad en Docker de forma integral: usuarios no-root, escaneo de vulnerabilidades, gestion correcta de secretos, capabilities/seccomp para hardening de bajo nivel, y evaluacion de confiabilidad de imagenes base (cadena de suministro).

**Nivel esperado:** Mid a Senior.

## Temas

| Orden | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Usuario No-Root en Containers | Pendiente | [[01 - Usuario No-Root en Containers]] |
| 2 | Escaneo de Vulnerabilidades de Imagenes | Pendiente | [[02 - Escaneo de Vulnerabilidades de Imagenes]] |
| 3 | Secrets Management | Pendiente | [[03 - Secrets Management]] |
| 4 | Capabilities y Seccomp | Pendiente | [[04 - Capabilities y Seccomp]] |
| 5 | Imagenes de Fuentes Confiables | Pendiente | [[05 - Imagenes de Fuentes Confiables]] |

## Como cerrar este capitulo

- [ ] Lei cada tema como capitulo, no solo como checklist.
- [ ] Configure un usuario no-root en un Dockerfile real.
- [ ] Escanee una imagen con Trivy o Docker Scout e interprete el reporte.
- [ ] Puedo explicar por que un secreto en `ARG` queda expuesto en el historial de la imagen.
- [ ] Tengo notas propias de dudas, decisiones o patrones que quiero repetir.
