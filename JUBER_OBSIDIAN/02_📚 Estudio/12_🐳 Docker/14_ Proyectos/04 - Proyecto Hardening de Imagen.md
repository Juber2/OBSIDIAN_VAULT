---
tags:
  - Docker
  - DevOps
  - Docker-Proyectos
Creado: 2026-07-28
Relacionado:
  - Proyectos
  - DOCKER
---

# 04 - Proyecto Hardening de Imagen

## Que vas a construir

Un ejercicio de **hardening completo** sobre una imagen existente (la de los proyectos anteriores), aplicando de punta a punta los capitulos de Seguridad y Optimizacion de Imagenes.

## Alcance funcional

- Imagen construida con usuario no-root, UID fijo y conocido.
- Capabilities reducidas a las minimas necesarias (`--cap-drop=ALL` con `--cap-add` selectivo).
- Imagen base evaluada entre Alpine, slim y distroless, con justificacion de la eleccion final.
- Imagen fijada por digest exacto en el Dockerfile o en la configuracion de despliegue.
- Escaneo de vulnerabilidades documentado con el resultado antes y despues del hardening.

## Checklist tecnica

- [ ] El proceso principal corre como usuario no-root con UID fijo documentado.
- [ ] Se verifica con `--cap-drop=ALL` que capabilities son realmente necesarias, agregando solo esas.
- [ ] Se documenta la comparacion de tamaño y compatibilidad entre Alpine, slim y distroless para este caso especifico.
- [ ] La imagen final esta fijada por digest, no solo por tag mutable.
- [ ] Un escaneo de vulnerabilidades muestra una reduccion medible de hallazgos criticos/altos respecto a la version sin hardening.

## Criterio de exito

El proyecto se considera completo cuando existe documentacion clara (un mini-ADR) de cada decision de hardening tomada, con evidencia medida (escaneo antes/despues, tamaño antes/despues) que demuestra el impacto real de cada cambio, no solo la aplicacion mecanica de una checklist.

## Errores comunes a evitar

- Aplicar hardening sin medir el impacto real (escaneo antes/despues), perdiendo la evidencia de que efectivamente mejoro la seguridad.
- Elegir distroless sin haber preparado una estrategia de debugging alternativa para cuando algo falle en produccion.
- No documentar el UID fijo usado, complicando la configuracion de volumenes externos con permisos correctos.

## Extension opcional

Investigar y documentar como se veria un perfil seccomp custom mas restrictivo que el default de Docker para este servicio especifico.
