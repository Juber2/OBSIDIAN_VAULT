---
tags:
  - Docker
  - DevOps
  - Docker-Seguridad
Creado: 2026-07-28
Relacionado:
  - Seguridad
  - DOCKER
---

# 02 - Escaneo de Vulnerabilidades de Imagenes

## Que vas a aprender

En este capitulo vas a aprender a **escanear imagenes Docker en busca de vulnerabilidades conocidas** con herramientas como Trivy y Docker Scout, integrando este escaneo como parte regular del ciclo de desarrollo, no como una revision aislada.

## Explicacion

Una imagen Docker incluye no solo el codigo de tu aplicacion, sino tambien un sistema operativo base y sus paquetes (Alpine, Debian, etc.), ademas de las dependencias de tu aplicacion (paquetes npm, pip, etc.). Cualquiera de esas capas puede tener **vulnerabilidades conocidas** (CVEs) documentadas en bases de datos publicas. Herramientas como **Trivy** (open source, muy usado) o **Docker Scout** (integrado en Docker) analizan una imagen construida y reportan que vulnerabilidades conocidas contiene, con su nivel de severidad.

El valor real de estas herramientas aparece cuando se integran al **pipeline de CI**, fallando el build (o al menos alertando) ante vulnerabilidades criticas nuevas, en vez de depender de que alguien las revise manualmente de forma esporadica.

## Modelo mental

Pensa el escaneo de vulnerabilidades como un control de sanidad antes de que un producto salga a la venta: revisa cada ingrediente (cada capa y paquete) contra una lista actualizada de problemas conocidos, y te avisa si algo con lo que estas por enviar al cliente tiene un defecto documentado.

## Ejemplo base

```bash
# Escanear una imagen local con Trivy
trivy image mi-imagen:latest

# Filtrar solo vulnerabilidades criticas y altas
trivy image --severity CRITICAL,HIGH mi-imagen:latest

# Docker Scout, integrado directamente en el CLI de Docker
docker scout cves mi-imagen:latest
```

## Como se usa en un proyecto real

1. Se integra el escaneo de vulnerabilidades como un paso del pipeline de CI, despues de construir la imagen y antes de publicarla.
2. Se define un umbral de severidad que bloquea el pipeline (por ejemplo, cualquier vulnerabilidad CRITICAL sin parche disponible) versus lo que solo genera una alerta informativa.
3. Se prioriza actualizar la imagen base a versiones parcheadas como primera linea de defensa, dado que muchas vulnerabilidades vienen del sistema operativo base, no del codigo propio.
4. Se re-escanean imagenes ya publicadas periodicamente, dado que nuevas vulnerabilidades se descubren en paquetes que no cambiaron desde el build original.
5. Se documenta el proceso de respuesta ante una vulnerabilidad critica descubierta en una imagen ya desplegada en produccion.

## Criterio junior

Un junior debe poder ejecutar un escaneo basico de una imagen con Trivy o Docker Scout e interpretar el reporte de severidades.

## Criterio mid

Un mid debe integrar el escaneo en el pipeline de CI con un umbral de severidad que bloquee builds problematicos, y priorizar actualizaciones de imagen base ante vulnerabilidades del sistema operativo.

## Criterio senior

Un senior debe definir la politica de gestion de vulnerabilidades del equipo (umbrales, re-escaneo periodico de imagenes ya desplegadas, proceso de respuesta ante hallazgos criticos) y balancear la friccion de bloquear pipelines contra el riesgo real de cada vulnerabilidad.

## Errores comunes

- No escanear imagenes en absoluto, descubriendo vulnerabilidades solo si aparecen en una auditoria externa o un incidente.
- Bloquear el pipeline ante cualquier vulnerabilidad sin importar severidad, generando tanta friccion que el equipo termina ignorando las alertas.
- No re-escanear imagenes ya publicadas, perdiendo visibilidad de vulnerabilidades descubiertas despues del build original.
- Enfocarse solo en vulnerabilidades del codigo propio, ignorando las que vienen de la imagen base del sistema operativo.

## Practica

- [ ] Escanea una imagen de un proyecto de practica con Trivy o Docker Scout.
- [ ] Filtra el reporte por severidad y identifica las vulnerabilidades criticas.
- [ ] Integra el escaneo como un paso de un pipeline de CI de ejemplo.
- [ ] Define un umbral de severidad que bloquearia el pipeline en tu proyecto.
- [ ] Investiga como re-escanearias periodicamente imagenes ya publicadas en un registry.

## Preguntas de repaso

- Que tipo de vulnerabilidades puede detectar un escaneo de imagen ademas de las del codigo propio de la aplicacion?
- Por que conviene integrar el escaneo en CI en vez de hacerlo solo manualmente de vez en cuando?
- Que umbral de severidad usarias para bloquear un pipeline, y por que no bloquear ante cualquier hallazgo?
- Por que es importante re-escanear imagenes ya publicadas y no solo escanear en el momento del build?
