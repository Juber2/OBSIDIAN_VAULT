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

# 03 - Secrets Management

## Que vas a aprender

En este capitulo vas a aprender las formas correctas de manejar **secretos** (contraseñas, API keys, certificados) en Docker, evitando los errores comunes que los dejan expuestos en la imagen o en el historial de build.

## Explicacion

Nunca se deben hardcodear secretos en el Dockerfile ni pasarlos como `ARG`/`ENV` sin cuidado: cualquier valor puesto en una instruccion `RUN` o `ARG` **queda visible en el historial de capas de la imagen** (`docker history`), incluso si un paso posterior "lo borra", porque esa capa anterior sigue existiendo. La forma correcta durante el **build** es usar **BuildKit secrets** (`--secret`), que monta el secreto temporalmente solo durante la ejecucion de una instruccion especifica, sin dejarlo en ninguna capa de la imagen final.

Para secretos usados en **runtime** (cuando el container ya esta corriendo), las opciones apropiadas son: variables de entorno inyectadas desde un vault o el orquestador (nunca hardcodeadas en `docker-compose.yml` versionado), Docker Swarm secrets si se usa Swarm, o el mecanismo de secrets nativo del orquestador (Kubernetes Secrets, AWS Secrets Manager, etc.) cuando se usa uno.

## Modelo mental

Pensa cada capa de una imagen como una pagina de un diario que, una vez escrita, no se puede arrancar: si en algun momento escribiste un secreto en una pagina, tacharlo en una pagina posterior no borra lo que ya quedo escrito antes. La unica forma segura es nunca escribirlo en ninguna pagina permanente del diario en primer lugar.

## Ejemplo base

```dockerfile
# BuildKit secret: el token nunca queda en ninguna capa de la imagen
# syntax=docker/dockerfile:1
FROM node:20-alpine
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token) npm install
```

```bash
docker build --secret id=npm_token,src=./npm_token.txt -t mi-imagen .
```

```yaml
# Runtime: variable de entorno inyectada, nunca hardcodeada en el YAML versionado
services:
  api:
    environment:
      DB_PASSWORD: ${DB_PASSWORD}   # viene de .env NO versionado o de un vault externo
```

## Como se usa en un proyecto real

1. Se usa `--secret` de BuildKit para cualquier credencial necesaria solo durante el build (tokens de registries privados, credenciales de paquetes privados).
2. Nunca se usan `ARG` o `ENV` en el Dockerfile para pasar secretos, dado que ambos quedan visibles en el historial de la imagen.
3. En runtime, los secretos se inyectan desde un vault (HashiCorp Vault, AWS Secrets Manager) o el mecanismo nativo del orquestador, nunca hardcodeados en archivos versionados.
4. Se audita periodicamente el historial de imagenes existentes (`docker history --no-trunc`) en busca de secretos filtrados en builds pasados.
5. Se documenta el proceso de rotacion de secretos ante una sospecha de filtracion, incluyendo reconstruir imagenes que pudieran contener el valor comprometido.

## Criterio junior

Un junior debe entender por que un secreto puesto en `ARG` o `ENV` en el Dockerfile queda expuesto en el historial de la imagen, incluso si se "borra" despues.

## Criterio mid

Un mid debe usar BuildKit secrets para credenciales necesarias durante el build, y manejar secretos de runtime via variables de entorno inyectadas externamente, nunca versionadas.

## Criterio senior

Un senior debe diseñar la estrategia completa de gestion de secretos del equipo (vault, rotacion, auditoria de imagenes existentes por filtraciones pasadas) y establecer el proceso de respuesta ante una sospecha de secreto comprometido.

## Errores comunes

- Poner un secreto en `ARG` o `ENV` del Dockerfile, asumiendo erroneamente que un paso posterior que lo "borra" lo elimina del historial de la imagen.
- Versionar un `docker-compose.yml` con secretos reales hardcodeados en `environment:`.
- No auditar imagenes existentes por secretos filtrados en builds pasados antes de adoptar BuildKit secrets.
- No tener un proceso de rotacion ante una sospecha de filtracion, dejando el secreto comprometido activo indefinidamente.

## Practica

- [ ] Intenta pasar un secreto via `ARG` y verifica con `docker history --no-trunc` que queda visible.
- [ ] Reescribe ese mismo caso usando BuildKit secrets (`--secret`) y verifica que ya no aparece en el historial.
- [ ] Audita un `docker-compose.yml` existente en busca de secretos hardcodeados.
- [ ] Investiga como inyectarias secretos de runtime desde un vault en tu infraestructura actual.
- [ ] Documenta el proceso de rotacion que aplicarias ante una sospecha de secreto comprometido.

## Preguntas de repaso

- Por que un secreto puesto en `ARG` sigue expuesto en el historial de la imagen aunque una instruccion posterior lo elimine del filesystem?
- Que ventaja da `--secret` de BuildKit sobre `ARG`/`ENV` para credenciales de build?
- Donde deberian vivir los secretos de runtime, y donde nunca deberian vivir?
- Que pasos incluirias en un proceso de rotacion ante un secreto comprometido?
