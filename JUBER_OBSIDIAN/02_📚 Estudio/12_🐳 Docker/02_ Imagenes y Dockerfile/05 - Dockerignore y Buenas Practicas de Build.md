---
tags:
  - Docker
  - DevOps
  - Docker-Imagenes
Creado: 2026-07-28
Relacionado:
  - Imagenes y Dockerfile
  - DOCKER
---

# 05 - Dockerignore y Buenas Practicas de Build

## Que vas a aprender

En este capitulo vas a aprender a usar **`.dockerignore`** para controlar el contexto de build, y a consolidar un checklist de buenas practicas generales para escribir Dockerfiles limpios y eficientes.

## Explicacion

Cuando corres `docker build`, Docker envia todo el contenido del directorio (el **contexto de build**) al daemon antes de empezar, incluso archivos que nunca vas a usar en el Dockerfile. Sin un `.dockerignore`, esto tipicamente incluye `.git`, `node_modules`, archivos de log, y otros artefactos pesados o innecesarios, ralentizando el build y potencialmente filtrando archivos sensibles (como `.env`) dentro de la imagen si se usa `COPY .` sin cuidado.

`.dockerignore` funciona igual que `.gitignore`: define patrones de archivos y carpetas que Docker excluye del contexto de build por completo, antes de que cualquier instruccion los pueda tocar.

## Modelo mental

Pensa el contexto de build como la mochila que llevas a upa antes de empezar a cocinar: cuanto mas peso innecesario metas ahi (archivos que nunca vas a usar), mas lento es cada viaje, y encima corres el riesgo de que algo que no deberia estar en la mochila (un secreto) termine mezclado con lo que si necesitas.

## Ejemplo base

```
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
.env
.env.*
dist
coverage
*.md
Dockerfile
.dockerignore
```

## Como se usa en un proyecto real

1. Se crea `.dockerignore` desde el inicio de cualquier proyecto Docker, antes del primer build, no como correccion posterior.
2. Se excluyen explicitamente archivos de secretos (`.env`) del contexto de build, para que ni siquiera puedan copiarse por accidente con un `COPY .` amplio.
3. Se excluye `node_modules` (o equivalente) del contexto, dejando que se genere dentro del container con `npm ci`, evitando arrastrar dependencias instaladas en el sistema operativo del host que pueden no ser compatibles.
4. Se revisa periodicamente el `.dockerignore` a medida que el proyecto agrega nuevos artefactos generados (carpetas de build, cache de test) que no deberian formar parte del contexto.
5. Se combina con buenas practicas generales: usar imagenes base oficiales y minimas, fijar versiones, minimizar el numero de capas sin sacrificar legibilidad, y no correr como root (ver capitulo de Seguridad).

## Criterio junior

Un junior debe poder crear un `.dockerignore` basico que excluya `node_modules`, `.git` y archivos de entorno.

## Criterio mid

Un mid debe auditar el contexto de build de un proyecto existente e identificar archivos sensibles o pesados que deberian excluirse.

## Criterio senior

Un senior debe establecer el `.dockerignore` y las buenas practicas de Dockerfile como parte del estandar de plantillas del equipo, revisando en code review que se respeten en cualquier proyecto nuevo.

## Errores comunes

- No tener `.dockerignore`, enviando todo el repositorio (incluyendo `.git` completo) como contexto de build en cada `docker build`.
- Dejar `.env` sin excluir, arriesgando que un `COPY .` amplio termine copiando secretos dentro de la imagen.
- Copiar `node_modules` del host en vez de instalarlo dentro del container, arriesgando incompatibilidades de arquitectura o sistema operativo.
- No revisar el `.dockerignore` a medida que el proyecto crece, dejando pasar nuevos artefactos pesados sin excluir.

## Practica

- [ ] Crea un `.dockerignore` completo para un proyecto de practica.
- [ ] Verifica que `.env` y `node_modules` esten excluidos del contexto de build.
- [ ] Mide el tamaño del contexto de build antes y despues de agregar el `.dockerignore`.
- [ ] Audita un Dockerfile existente contra un checklist de buenas practicas generales.
- [ ] Documenta el `.dockerignore` estandar que usarias como plantilla para nuevos proyectos.

## Preguntas de repaso

- Que es el contexto de build y por que un `.dockerignore` lo afecta directamente?
- Que riesgo de seguridad concreto previene excluir `.env` del contexto de build?
- Por que conviene instalar dependencias dentro del container en vez de copiar `node_modules` del host?
- Que impacto tiene un `.dockerignore` bien configurado en la velocidad de los builds?
