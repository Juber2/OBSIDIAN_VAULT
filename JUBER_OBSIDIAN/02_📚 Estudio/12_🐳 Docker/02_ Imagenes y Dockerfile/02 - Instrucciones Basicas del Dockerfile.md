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

# 02 - Instrucciones Basicas del Dockerfile

## Que vas a aprender

En este capitulo vas a dominar las instrucciones fundamentales de un **Dockerfile**: `FROM`, `WORKDIR`, `COPY`, `RUN`, `ENV`, `EXPOSE`, `CMD` y `ENTRYPOINT`, y la diferencia clave entre estos dos ultimos.

## Explicacion

`FROM` define la imagen base de partida. `WORKDIR` establece el directorio de trabajo dentro del container (equivalente a `cd`, pero persistente para las instrucciones siguientes). `COPY` trae archivos del contexto de build al filesystem de la imagen. `RUN` ejecuta un comando durante el **build** (crea una capa nueva). `ENV` define variables de entorno disponibles tanto en build como en runtime del container.

La distincion mas confundida es `CMD` vs `ENTRYPOINT`: `ENTRYPOINT` define el ejecutable fijo del container (lo que siempre corre), mientras `CMD` define los argumentos por defecto que se le pasan (y que pueden sobreescribirse facilmente al correr `docker run imagen otro-comando`). Cuando se usan juntos, `ENTRYPOINT` es el comando y `CMD` son sus argumentos default.

## Modelo mental

Pensa `ENTRYPOINT` como el verbo fijo de la oracion (lo que el container siempre "hace"), y `CMD` como el complemento que puede cambiar segun el contexto. Si el Dockerfile dice `ENTRYPOINT ["node"]` y `CMD ["server.js"]`, el container por defecto corre `node server.js`, pero un `docker run imagen otro.js` cambia el complemento sin tocar el verbo.

## Ejemplo base

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
ENV NODE_ENV=production
EXPOSE 3000
ENTRYPOINT ["node"]
CMD ["dist/main.js"]
```

```bash
docker run mi-imagen              # ejecuta: node dist/main.js
docker run mi-imagen otro.js      # ejecuta: node otro.js (CMD sobreescrito)
```

## Como se usa en un proyecto real

1. `FROM` siempre especifica una version exacta o un digest, nunca `latest`, para builds reproducibles.
2. `COPY` de archivos de dependencias (`package.json`) se hace antes que `COPY .` del resto del codigo, para aprovechar cache (ver siguiente tema).
3. `ENTRYPOINT` se usa para el ejecutable principal cuando se quiere que el container siempre corra ese proceso, incluso si alguien pasa argumentos distintos.
4. `EXPOSE` documenta el puerto que la aplicacion usa, aunque no lo publica realmente (eso lo hace `-p` en `docker run`); sirve como documentacion y para herramientas que la leen automaticamente.
5. Se evita usar `ENV` para secretos (quedan visibles en `docker inspect`); para eso se usan mecanismos de secrets (ver capitulo de Seguridad).

## Criterio junior

Un junior debe poder escribir un Dockerfile basico funcional con `FROM`, `WORKDIR`, `COPY`, `RUN` y `CMD`.

## Criterio mid

Un mid debe usar `ENTRYPOINT` y `CMD` combinados de forma correcta, y estructurar el Dockerfile pensando en el orden de instrucciones.

## Criterio senior

Un senior debe definir el estandar de Dockerfiles del equipo (estructura, convenciones de `ENTRYPOINT`/`CMD`, manejo de variables de entorno) y revisar en code review el uso correcto de cada instruccion.

## Errores comunes

- Confundir `CMD` con `ENTRYPOINT` y sorprenderse cuando pasar un argumento a `docker run` no hace lo esperado.
- Usar `FROM imagen:latest` en vez de una version fijada, arriesgando builds no reproducibles cuando la imagen base cambia.
- Poner secretos en `ENV`, exponiendolos en texto plano via `docker inspect`.
- No usar `WORKDIR` y depender de rutas absolutas repetidas en cada instruccion, generando un Dockerfile fragil.

## Practica

- [ ] Escribe un Dockerfile basico con las instrucciones fundamentales para una aplicacion simple.
- [ ] Prueba combinar `ENTRYPOINT` y `CMD`, y sobreescribe `CMD` al correr el container.
- [ ] Fija la version exacta de la imagen base en vez de usar `latest`.
- [ ] Identifica si tu Dockerfile actual tiene algun secreto expuesto en `ENV`.
- [ ] Documenta el proposito de `EXPOSE` en tu Dockerfile aunque no publique el puerto por si solo.

## Preguntas de repaso

- Que diferencia hay entre `CMD` y `ENTRYPOINT`, y que pasa cuando se usan juntos?
- Por que fijar la version de la imagen base es mejor que usar `latest`?
- Por que `ENV` no es un lugar seguro para guardar secretos?
- Que hace realmente `EXPOSE` y que no hace?
