---
tags:
  - Docker
  - DevOps
  - Docker-Optimizacion
Creado: 2026-07-28
Relacionado:
  - Optimizacion de Imagenes
  - DOCKER
---

# 01 - Tamaño de Imagen y Capas

## Que vas a aprender

En este capitulo vas a aprender por que el **tamaño de una imagen** importa mas alla de lo obvio (espacio en disco), y las tecnicas basicas para reducirlo: menos capas innecesarias, limpieza dentro de la misma instruccion `RUN`, y eleccion consciente de que se incluye.

## Explicacion

El tamaño de una imagen impacta directamente en: tiempo de `docker pull` (relevante en cada deploy y en auto-escalado, donde una instancia nueva no puede recibir trafico hasta descargar la imagen), espacio en disco de los servidores, tiempo de transferencia en CI/CD, y superficie de ataque (mas paquetes instalados es mas codigo potencialmente vulnerable).

Una tecnica clave es limpiar archivos temporales **dentro de la misma instruccion `RUN`** que los genero, no en una instruccion posterior: como cada `RUN` crea su propia capa, si instalas paquetes y limpias cache en dos `RUN` separados, el peso de la cache sigue existiendo en la capa anterior aunque la instruccion siguiente la haya borrado del filesystem final visible.

## Modelo mental

Pensa cada instruccion `RUN` como una hoja de papel que se archiva permanentemente una vez escrita. Si ensucias la hoja (instalas paquetes con cache temporal) y despues, en una hoja nueva, "limpias el escritorio", la hoja sucia original sigue archivada tal cual quedo, ocupando espacio en el archivo entero (la imagen), aunque el escritorio final se vea limpio.

## Ejemplo base

```dockerfile
# Mal: dos capas, la cache de apt queda archivada en la primera aunque se borre en la segunda
RUN apt-get update && apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# Bien: una sola capa, sin dejar cache atras
RUN apt-get update && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
```

## Como se usa en un proyecto real

1. Se combinan instrucciones relacionadas de instalacion y limpieza en un unico `RUN` con `&&`, evitando que la cache quede archivada en una capa intermedia.
2. Se prefieren imagenes base minimas (Alpine, slim, distroless, ver siguiente tema) cuando la aplicacion lo permite.
3. Se instalan solo los paquetes del sistema operativo estrictamente necesarios, evitando `apt-get install` de herramientas de conveniencia que no se usan en produccion.
4. Se mide el tamaño de la imagen con `docker images` regularmente, como parte de la revision de cambios al Dockerfile.
5. Se combina esto con multi-stage builds (ya visto) como la tecnica de mayor impacto para reducir tamaño en la imagen final.

## Criterio junior

Un junior debe entender por que limpiar cache en la misma instruccion `RUN` que la genero reduce el tamaño de la imagen, a diferencia de limpiarla en una instruccion posterior.

## Criterio mid

Un mid debe optimizar Dockerfiles existentes combinando instrucciones relacionadas y eligiendo imagenes base mas livianas segun las necesidades reales del proyecto.

## Criterio senior

Un senior debe establecer estandares de tamaño de imagen para el equipo, revisando en code review Dockerfiles que agreguen peso innecesario, y midiendo el impacto real de optimizaciones en tiempos de deploy y auto-escalado.

## Errores comunes

- Limpiar cache o archivos temporales en una instruccion `RUN` separada de la que los genero, sin reducir realmente el tamaño de la imagen.
- Instalar herramientas de conveniencia (editores, utilidades de debugging) en la imagen de produccion sin necesidad real.
- No medir el tamaño de la imagen de forma regular, perdiendo visibilidad de como crece con el tiempo.
- Asumir que una imagen "funciona bien" significa que esta bien optimizada, sin revisar su composicion real.

## Practica

- [ ] Compara el tamaño de una imagen antes y despues de combinar instrucciones de instalacion y limpieza en un unico `RUN`.
- [ ] Revisa `docker history` de una imagen existente e identifica capas con cache no limpiada correctamente.
- [ ] Elimina paquetes de conveniencia innecesarios de un Dockerfile de produccion.
- [ ] Mide el tamaño final de una imagen con `docker images` despues de optimizarla.
- [ ] Anota una checklist de optimizacion de tamaño para aplicar a cualquier Dockerfile nuevo.

## Preguntas de repaso

- Por que limpiar cache en una instruccion `RUN` separada de la que la genero no reduce el tamaño real de la imagen?
- Que impacto tiene el tamaño de una imagen en el tiempo de auto-escalado de un sistema?
- Que criterio usarias para decidir si un paquete del sistema operativo es realmente necesario en la imagen de produccion?
- Como medirias el tamaño real de una imagen y sus capas individuales?
