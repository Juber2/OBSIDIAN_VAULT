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

# 04 - Multi-Arch Builds con Buildx

## Que vas a aprender

En este capitulo vas a aprender a construir imagenes **multi-arquitectura** (x86_64 y ARM64) con `docker buildx`, necesario desde que servidores ARM (AWS Graviton, chips Apple Silicon en desarrollo local) se volvieron comunes.

## Explicacion

Una imagen Docker se construye para una arquitectura de CPU especifica. Si construis una imagen en una laptop con chip Apple Silicon (ARM64) y la desplegas en un servidor x86_64 sin haber construido para esa arquitectura, la imagen simplemente no corre (o corre con emulacion muy lenta). `docker buildx` permite construir una **imagen manifest multi-arquitectura**: un unico tag que en realidad apunta a variantes distintas segun la arquitectura del host que hace el pull, resuelto automaticamente por Docker.

Esto es cada vez mas relevante porque el desarrollo local en Mac (Apple Silicon, ARM64) es comun, mientras que muchos servidores de produccion siguen siendo x86_64 (aunque instancias ARM como AWS Graviton estan creciendo por su mejor relación costo/performance).

## Modelo mental

Pensa una imagen multi-arch como una etiqueta de producto que en realidad representa varias versiones fisicas distintas del mismo producto (una para cada tipo de "enchufe" de pared), y el sistema (Docker) automaticamente te entrega la version correcta segun el enchufe que tenga tu casa (la arquitectura de tu servidor), sin que vos tengas que elegir manualmente.

## Ejemplo base

```bash
# Crear un builder que soporte multi-arquitectura
docker buildx create --use --name multi-builder

# Construir y publicar para ambas arquitecturas en un unico tag
docker buildx build --platform linux/amd64,linux/arm64 \
  -t mi-registro/mi-imagen:1.0 --push .
```

## Como se usa en un proyecto real

1. Se configura `buildx` en el pipeline de CI para publicar imagenes multi-arquitectura desde el inicio del proyecto, evitando problemas sorpresivos al desplegar en servidores de arquitectura distinta a la de desarrollo.
2. Se verifica que todas las dependencias de la aplicacion (especialmente paquetes con codigo nativo compilado) tengan soporte para ambas arquitecturas antes de asumir que el build multi-arch funcionara sin problemas.
3. Se mide el tiempo de build multi-arch, dado que compilar para una arquitectura distinta a la del host de build puede requerir emulacion (mas lenta) a menos que se use un builder nativo remoto para cada arquitectura.
4. Se documenta que arquitecturas soporta oficialmente cada imagen publicada por el equipo.
5. Se evita asumir que "funciona en mi maquina" (ARM64 en desarrollo) garantiza que funcionara igual en produccion (x86_64) sin haber probado esa arquitectura especifica.

## Criterio junior

Un junior debe entender que una imagen se construye para una arquitectura especifica y que multi-arch resuelve la incompatibilidad entre desarrollo (posiblemente ARM64) y produccion (posiblemente x86_64).

## Criterio mid

Un mid debe configurar `buildx` para construir y publicar imagenes multi-arquitectura como parte del flujo estandar de CI/CD.

## Criterio senior

Un senior debe decidir la estrategia de arquitecturas soportadas por la organizacion (incluyendo evaluar migrar cargas a instancias ARM por costo/performance) y asegurar que el pipeline de build multi-arch sea eficiente y confiable.

## Errores comunes

- Construir y probar una imagen solo en la arquitectura de la maquina de desarrollo, sin verificar que funcione en la arquitectura real de produccion.
- No verificar que dependencias con codigo nativo compilado tengan soporte para ambas arquitecturas antes de asumir que un build multi-arch funcionara sin cambios.
- Subestimar el tiempo de build multi-arch quando se depende de emulacion en vez de builders nativos remotos por arquitectura.
- No documentar que arquitecturas soporta oficialmente una imagen publicada, generando confusion en equipos que la consumen.

## Practica

- [ ] Configura un builder de `buildx` con soporte multi-arquitectura.
- [ ] Construye una imagen para `linux/amd64` y `linux/arm64` simultaneamente.
- [ ] Verifica que alguna dependencia nativa de tu proyecto (si aplica) sea compatible con ambas arquitecturas.
- [ ] Mide el tiempo de build multi-arch comparado con un build de una sola arquitectura.
- [ ] Documenta que arquitecturas soportaria oficialmente una imagen de tu proyecto.

## Preguntas de repaso

- Que pasa si intentas correr una imagen construida solo para ARM64 en un servidor x86_64 sin soporte multi-arch?
- Que resuelve una imagen manifest multi-arquitectura para el usuario final que hace `docker pull`?
- Por que el desarrollo en Mac con Apple Silicon hizo mas relevante el soporte multi-arch en los ultimos años?
- Que riesgo especifico introducen las dependencias con codigo nativo compilado en un build multi-arch?
