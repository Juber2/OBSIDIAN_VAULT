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

# 05 - Analisis de Imagenes con Dive

## Que vas a aprender

En este capitulo vas a aprender a usar **Dive**, una herramienta que permite explorar visualmente el contenido de cada capa de una imagen Docker, identificando exactamente que archivos ocupan mas espacio y donde hay desperdicio recuperable.

## Explicacion

Mientras `docker history` da una vista general de cuanto peso agrega cada instruccion, **Dive** permite navegar capa por capa el contenido real del filesystem, viendo que archivos especificos existen en cada una, y calcula un **"efficiency score"** que estima cuanto espacio se podria recuperar eliminando archivos duplicados o desperdiciados entre capas (por ejemplo, un archivo grande creado en una capa y luego modificado ligeramente en otra, dejando ambas versiones ocupando espacio).

Esta herramienta es particularmente util para diagnosticar por que una imagen es mas pesada de lo esperado cuando las tecnicas basicas (multi-stage, limpieza en el mismo `RUN`) ya se aplicaron y el tamaño sigue sin bajar lo suficiente.

## Modelo mental

Pensa Dive como una lupa de alta resolucion sobre cada capa de la torta de imagen (ver capitulo de Anatomia de Imagenes): en vez de ver solo el peso total de cada rebanada, podes ver exactamente que ingredientes especificos hay en cada una y cuales se repiten innecesariamente entre rebanadas distintas.

## Ejemplo base

```bash
dive mi-imagen:latest
# Abre una interfaz interactiva en terminal para explorar capas y archivos

# Modo CI: falla el build si el efficiency score esta por debajo de un umbral
dive mi-imagen:latest --ci --lowestEfficiency 0.9
```

## Como se usa en un proyecto real

1. Se usa Dive de forma exploratoria cuando una imagen resulta mas pesada de lo esperado, para identificar la capa y el archivo especifico responsable.
2. Se integra `dive --ci` en el pipeline como gate opcional, fallando el build si el efficiency score cae por debajo de un umbral acordado por el equipo.
3. Se usa para verificar visualmente que un multi-stage build realmente esta dejando atras lo que se espera, confirmando que la etapa final no arrastra archivos de la etapa de build.
4. Se documenta cualquier hallazgo relevante de Dive (por ejemplo, un archivo grande duplicado entre capas) como una mejora concreta a aplicar en el Dockerfile.
5. Se complementa con `docker history` para una primera vista rapida, reservando Dive para investigaciones mas profundas.

## Criterio junior

Un junior debe poder instalar y correr Dive sobre una imagen para explorar su contenido capa por capa de forma interactiva.

## Criterio mid

Un mid debe interpretar el efficiency score de Dive y usarlo para identificar oportunidades concretas de optimizacion en un Dockerfile existente.

## Criterio senior

Un senior debe integrar Dive (o una herramienta equivalente) como gate automatizado de calidad de imagenes en el pipeline de CI, definiendo el umbral de efficiency score aceptable para el equipo.

## Errores comunes

- Optimizar una imagen a ciegas sin usar una herramienta como Dive para confirmar donde realmente esta el desperdicio.
- No integrar Dive (o equivalente) en CI, dependiendo solo de revisiones manuales esporadicas del tamaño de imagen.
- Perseguir un efficiency score perfecto sin considerar el costo de tiempo de esa optimizacion frente al beneficio real obtenido.
- No usar Dive para verificar que un multi-stage build realmente esta dejando atras los archivos esperados de la etapa de build.

## Practica

- [ ] Instala Dive y explora una imagen de un proyecto de practica capa por capa.
- [ ] Identifica el efficiency score de esa imagen y que capas contribuyen mas al desperdicio.
- [ ] Aplica una optimizacion concreta basada en lo encontrado con Dive.
- [ ] Verifica con Dive que un multi-stage build no esta arrastrando archivos innecesarios de la etapa de build.
- [ ] Define un umbral de efficiency score que integrarias como gate en tu pipeline de CI.

## Preguntas de repaso

- Que ventaja da Dive sobre `docker history` para diagnosticar el tamaño de una imagen?
- Que mide el "efficiency score" de Dive?
- Como usarias Dive para verificar que un multi-stage build esta funcionando correctamente?
- Que consideraria antes de integrar Dive como gate obligatorio en un pipeline de CI?
