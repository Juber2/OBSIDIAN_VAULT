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

# 01 - Proyecto Dockerizar API con Multi-Stage

## Que vas a construir

Dockerizar una API existente (NestJS, .NET, o cualquier stack que ya tengas) aplicando de punta a punta los capitulos de Fundamentos, Imagenes/Dockerfile y Volumenes.

## Alcance funcional

- Dockerfile multi-stage con etapa de build separada de la etapa de produccion.
- `.dockerignore` completo excluyendo archivos innecesarios y sensibles.
- Usuario no-root configurado para el proceso principal.
- Volumen persistente para cualquier dato que la aplicacion necesite guardar entre reinicios.
- Imagen final medida y optimizada (comparada contra una version sin optimizar).

## Checklist tecnica

- [ ] La imagen final no contiene el compilador ni devDependencies/herramientas de build.
- [ ] `.dockerignore` excluye `.git`, `node_modules` o equivalente, y cualquier archivo `.env`.
- [ ] El proceso principal corre como usuario no-root verificado con `whoami` dentro del container.
- [ ] El orden de instrucciones del Dockerfile aprovecha correctamente el cache de build (dependencias antes que codigo fuente).
- [ ] Se documenta el tamaño de la imagen antes y despues de aplicar multi-stage build.
- [ ] Un volumen nombrado persiste datos correctamente entre `docker stop`/`docker run` sucesivos.

## Criterio de exito

El proyecto se considera completo cuando la imagen final es significativamente mas chica que una version de una sola etapa equivalente, corre como usuario no-root, y cualquier dato que la aplicacion necesite persistir sobrevive a la eliminacion y recreacion del container.

## Errores comunes a evitar

- Copiar todo el codigo fuente antes de instalar dependencias, perdiendo cache de build en cada cambio.
- Dejar el proceso corriendo como root sin justificacion.
- No verificar que los datos realmente persisten a traves de un volumen antes de dar el proyecto por terminado.

## Extension opcional

Agregar un `HEALTHCHECK` real en el Dockerfile que verifique un endpoint de salud de la aplicacion, como puente hacia el proyecto de stack full-stack mas adelante en este capitulo.
