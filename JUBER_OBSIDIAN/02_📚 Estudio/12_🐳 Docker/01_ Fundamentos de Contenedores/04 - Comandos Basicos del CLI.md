---
tags:
  - Docker
  - DevOps
  - Docker-Fundamentos
Creado: 2026-07-28
Relacionado:
  - Fundamentos de Contenedores
  - DOCKER
---

# 04 - Comandos Basicos del CLI

## Que vas a aprender

En este capitulo vas a dominar los comandos esenciales del dia a dia con Docker: `run`, `ps`, `exec`, `logs`, `stop`, `rm`, y las opciones mas usadas de cada uno.

## Explicacion

`docker run` crea y arranca un container a partir de una imagen, con opciones clave: `-d` (detached, en background), `-p` (mapeo de puertos), `-v` (montar volumenes), `-e` (variables de entorno), `--name` (nombrarlo para referenciarlo facil despues). `docker ps` lista containers en ejecucion (`-a` para incluir los detenidos). `docker exec` permite ejecutar un comando dentro de un container ya corriendo, la forma tipica de "entrar" a inspeccionarlo (`docker exec -it container bash`). `docker logs` muestra la salida estandar del proceso principal del container, esencial para debugging.

Entender la diferencia entre `stop` (envia SIGTERM, da tiempo a cerrar limpio) y `kill` (SIGKILL, corte inmediato) es importante para no perder datos en aplicaciones que necesitan un shutdown graceful.

## Modelo mental

Pensa cada comando como una herramienta especifica de una caja de herramientas del dia a dia: `run` es "encender la maquina", `ps` es "ver que maquinas estan prendidas", `exec` es "abrir una ventanita para mirar adentro sin apagarla", `logs` es "leer el diario de bitacora", y `stop` es "apagarla ordenadamente" en vez de "cortarle la luz de golpe" (`kill`).

## Ejemplo base

```bash
docker run -d --name mi-api -p 3000:3000 -e NODE_ENV=production mi-imagen:latest
docker ps                          # containers corriendo
docker ps -a                       # incluye detenidos
docker logs -f mi-api              # sigue el log en tiempo real
docker exec -it mi-api sh          # entra al container con una shell interactiva
docker stop mi-api                 # shutdown graceful (SIGTERM, luego SIGKILL si no responde)
docker rm mi-api                   # elimina el container detenido
```

## Como se usa en un proyecto real

1. Se nombra siempre los containers importantes (`--name`) para referenciarlos facilmente en scripts y comandos posteriores.
2. Se usa `docker logs -f` como primera herramienta de diagnostico ante cualquier comportamiento inesperado.
3. Se usa `docker exec -it container sh` (o `bash` si esta disponible) para inspeccionar el estado interno de un container corriendo, nunca para "arreglar" algo de forma permanente (los cambios se pierden al recrear el container).
4. Se prefiere `stop` sobre `kill` en cualquier flujo normal, reservando `kill` para containers realmente colgados que no responden a SIGTERM.
5. Se limpian containers detenidos periodicamente (`docker container prune`) para no acumular basura en el sistema.

## Criterio junior

Un junior debe poder correr, listar, ver logs, entrar y detener un container usando los comandos basicos del CLI.

## Criterio mid

Un mid debe usar las opciones avanzadas de cada comando (variables de entorno, mapeo de puertos, nombres) de forma fluida, y diagnosticar problemas basicos revisando logs y el estado del container.

## Criterio senior

Un senior debe automatizar el uso de estos comandos en scripts y pipelines, entendiendo bien las diferencias entre señales (SIGTERM vs SIGKILL) para diseñar shutdowns graceful en las aplicaciones que corren dentro de los containers.

## Errores comunes

- Usar `docker kill` como primera opcion en vez de `docker stop`, arriesgando corte abrupto de operaciones en curso.
- Modificar archivos dentro de un container corriendo via `exec` esperando que el cambio persista, sin entender que se pierde al recrear el container.
- No limpiar containers e imagenes viejas periodicamente, acumulando espacio en disco innecesariamente.
- Olvidar `-d` y bloquear la terminal con un container en primer plano cuando se queria correrlo en background.

## Practica

- [ ] Corre un container en background con nombre, puerto mapeado y una variable de entorno.
- [ ] Lista los containers corriendo y los detenidos.
- [ ] Sigue los logs de un container en tiempo real.
- [ ] Entra a un container corriendo con una shell interactiva e inspecciona un archivo.
- [ ] Detiene el container con `stop` y luego eliminalo con `rm`.

## Preguntas de repaso

- Que diferencia hay entre `docker stop` y `docker kill`?
- Que pasa con un cambio hecho dentro de un container via `exec` si el container se recrea despues?
- Para que sirve la opcion `-d` en `docker run`?
- Como verificarias el estado y los logs de un container que se comporta de forma inesperada?
