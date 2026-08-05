---
tags:
  - Docker
  - DevOps
  - Docker-Volumenes
Creado: 2026-07-28
Relacionado:
  - Volumenes y Persistencia
  - DOCKER
---

# 05 - Named Volumes en Produccion

## Que vas a aprender

En este capitulo vas a consolidar las practicas de uso de **volumenes nombrados en produccion**: drivers de volumen para almacenamiento distribuido, permisos y usuario dentro del container, y la relacion entre volumenes y alta disponibilidad.

## Explicacion

En un servidor unico, el driver `local` de volumenes alcanza, pero en un entorno con **multiples nodos/servidores** (necesario para alta disponibilidad real), un volumen `local` esta atado fisicamente a un solo servidor: si el container se reprograma en otro nodo, no encuentra sus datos. Para eso existen **drivers de volumen para almacenamiento distribuido** (NFS, EBS con anotaciones especificas de orquestador, drivers de proveedores cloud) que desacoplan el volumen de un servidor especifico.

Otro punto critico en produccion es la gestion de **permisos**: el proceso dentro del container corre con un UID especifico (idealmente no root, ver capitulo de Seguridad), y ese UID debe tener permisos correctos sobre los archivos del volumen, algo que suele causar errores confusos de "permission denied" cuando no se planifica desde el Dockerfile.

## Modelo mental

Pensa un volumen `local` como un cajon fijo atornillado a un escritorio especifico: perfecto si siempre trabajas en ese escritorio, pero inutil si te mudan a otro escritorio (otro servidor) y el cajon se quedo atras. Un volumen distribuido es como un cajon portatil que te sigue a cualquier escritorio de la oficina.

## Ejemplo base

```dockerfile
# Crear un usuario no-root con un UID especifico y asegurar permisos sobre el volumen
FROM node:20-alpine
RUN addgroup -g 1001 app && adduser -D -u 1001 -G app app
WORKDIR /app
COPY --chown=app:app . .
USER app
VOLUME /app/uploads
```

```bash
# El volumen debe tener permisos compatibles con el UID 1001 usado dentro del container
docker run -v uploads:/app/uploads mi-imagen
```

## Como se usa en un proyecto real

1. Se define un usuario no-root con un UID fijo y conocido en el Dockerfile, y se usa `--chown` en `COPY` para asegurar permisos correctos desde el inicio.
2. En clusters multi-nodo, se evalua un driver de volumen distribuido en vez de `local`, segun el orquestador y proveedor cloud usado.
3. Se documenta el UID/GID usado por cada imagen, para poder configurar permisos de volumenes externos (NFS u otros) de forma consistente.
4. Se separan claramente volumenes que deben ser compartidos entre multiples instancias (uploads de usuarios) de los que son exclusivos de una instancia (cache local descartable).
5. Se evita depender de volumenes locales para cualquier dato que deba sobrevivir a la reprogramacion de un container en otro nodo del cluster.

## Criterio junior

Un junior debe poder crear un usuario no-root en un Dockerfile y entender por que los permisos de un volumen pueden generar errores si no coinciden con el UID del proceso.

## Criterio mid

Un mid debe configurar correctamente permisos entre el UID del container y los volumenes que usa, diagnosticando errores de "permission denied" relacionados con esto.

## Criterio senior

Un senior debe diseñar la estrategia de almacenamiento persistente para un sistema desplegado en multiples nodos, eligiendo el driver de volumen apropiado y garantizando que ningun dato critico dependa de almacenamiento local no distribuido.

## Errores comunes

- Usar volumenes `local` para datos que deben sobrevivir a la reprogramacion de un container en un cluster multi-nodo, perdiendo acceso a esos datos.
- No fijar un UID conocido para el usuario no-root del container, generando problemas de permisos dificiles de reproducir entre entornos.
- Mezclar en el mismo volumen datos que deberian ser compartidos entre instancias con datos que deberian ser exclusivos de cada una.
- No documentar el UID/GID esperado por la imagen, complicando la configuracion de almacenamiento externo por parte de infraestructura.

## Practica

- [ ] Crea un Dockerfile con un usuario no-root de UID fijo y verifica permisos correctos sobre un volumen montado.
- [ ] Investiga que drivers de volumen distribuido soporta el orquestador o proveedor cloud que usarias en produccion.
- [ ] Diagnostica (o simula) un error de permisos entre el UID del container y un volumen externo.
- [ ] Documenta el UID/GID de una imagen de tu proyecto para referencia del equipo de infraestructura.
- [ ] Anota que datos de tu sistema necesitarian almacenamiento distribuido si escalaras a multiples nodos.

## Preguntas de repaso

- Por que un volumen `local` es insuficiente en un cluster con multiples nodos?
- Que problema de permisos puede surgir entre el UID del proceso dentro del container y un volumen externo?
- Que diferencia hay entre un dato que debe compartirse entre instancias y uno que debe ser exclusivo de cada una?
- Que informacion sobre una imagen es util documentar para facilitar la configuracion de almacenamiento externo?
