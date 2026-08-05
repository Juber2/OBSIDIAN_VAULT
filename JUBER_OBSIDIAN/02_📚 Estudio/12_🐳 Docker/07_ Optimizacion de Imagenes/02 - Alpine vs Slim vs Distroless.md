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

# 02 - Alpine vs Slim vs Distroless

## Que vas a aprender

En este capitulo vas a comparar las tres familias principales de imagenes base minimas: **Alpine** (basada en musl libc), **slim** (Debian reducido), y **distroless** (sin shell ni gestor de paquetes en absoluto), con sus tradeoffs reales.

## Explicacion

**Alpine** es extremadamente liviana (unos pocos MB) porque usa `musl` en vez de `glibc` como libreria estandar de C, lo cual la hace muy popular, pero puede causar incompatibilidades sutiles con paquetes compilados que esperan `glibc` (comunes en ciertas dependencias nativas de Node o Python). **Slim** son variantes de Debian con el minimo de paquetes, manteniendo compatibilidad total con `glibc` a cambio de un tamaño algo mayor que Alpine.

**Distroless** (de Google) va un paso mas alla: la imagen final no tiene shell, gestor de paquetes, ni casi ninguna herramienta, solo el runtime estrictamente necesario para ejecutar la aplicacion. Esto reduce drasticamente la superficie de ataque (un atacante que compromete el proceso no tiene ni siquiera una shell para moverse lateralmente), pero tambien elimina herramientas de debugging dentro del container, lo cual cambia como se diagnostican problemas en produccion.

## Modelo mental

Pensa Alpine como una casa muy pequeña y eficiente pero con instalaciones ligeramente distintas a las estandar (musl vs glibc), que a veces requieren adaptadores especiales. Slim es una casa de tamaño estandar pero sin muebles de mas. Distroless es un departamento sin puerta de entrada para visitas: extremadamente seguro, pero si algo se rompe adentro, no hay forma facil de entrar a arreglarlo desde adentro mismo.

## Ejemplo base

```dockerfile
# Alpine: muy liviana, cuidado con dependencias nativas incompatibles con musl
FROM node:20-alpine

# Slim: compatibilidad glibc completa, algo mas pesada
FROM node:20-slim

# Distroless: minima superficie de ataque, sin shell para debugging
FROM gcr.io/distroless/nodejs20-debian12
```

## Como se usa en un proyecto real

1. Se prueba Alpine primero por su tamaño, pero se verifica que todas las dependencias nativas de la aplicacion sean compatibles con `musl` antes de adoptarla en produccion.
2. Se usa slim cuando la aplicacion depende de paquetes que requieren `glibc` especificamente, o cuando la compatibilidad es mas critica que ahorrar los MB adicionales.
3. Se evalua distroless para servicios de alta sensibilidad de seguridad, aceptando el costo de debugging mas dificil (se depende de `docker exec` con herramientas externas o logs mas exhaustivos).
4. Se documenta la eleccion de imagen base y su justificacion, dado que cambiarla despues puede requerir revalidar compatibilidad de dependencias.
5. Se mide el tamaño final real de la imagen con cada opcion antes de decidir, en vez de asumir la diferencia sin datos.

## Criterio junior

Un junior debe conocer las tres opciones (Alpine, slim, distroless) y saber que Alpine puede tener incompatibilidades por usar `musl` en vez de `glibc`.

## Criterio mid

Un mid debe diagnosticar y resolver problemas de compatibilidad al migrar de slim a Alpine, y decidir cuando el ahorro de tamaño justifica el riesgo de incompatibilidad.

## Criterio senior

Un senior debe evaluar distroless para servicios criticos de seguridad, diseñando una estrategia de debugging alternativa (logging exhaustivo, herramientas externas) que compense la falta de shell dentro del container.

## Errores comunes

- Migrar a Alpine sin probar exhaustivamente las dependencias nativas de la aplicacion, descubriendo incompatibilidades recien en produccion.
- Elegir distroless sin haber preparado una estrategia de debugging alternativa, quedando "a ciegas" ante un problema en produccion.
- Asumir que Alpine siempre es la mejor opcion solo por ser la mas chica, sin considerar el costo de compatibilidad.
- No medir el tamaño real resultante de cada opcion para el caso especifico del proyecto antes de decidir.

## Practica

- [ ] Construye la misma aplicacion con imagen base Alpine y con slim, comparando tamaño y compatibilidad.
- [ ] Identifica si alguna dependencia de tu proyecto tiene problemas de compatibilidad con Alpine/musl.
- [ ] Investiga como debuggearias un container distroless sin shell disponible.
- [ ] Evalua si algun servicio de tu sistema seria buen candidato para distroless por su sensibilidad de seguridad.
- [ ] Documenta la eleccion de imagen base de tu proyecto actual y su justificacion.

## Preguntas de repaso

- Que diferencia tecnica fundamental hay entre Alpine y las imagenes basadas en Debian (slim)?
- Que riesgo de compatibilidad especifico introduce Alpine por usar `musl` en vez de `glibc`?
- Que ventaja de seguridad da distroless, y que costo de operabilidad trae a cambio?
- Como decidirias entre Alpine, slim y distroless para un servicio nuevo?
