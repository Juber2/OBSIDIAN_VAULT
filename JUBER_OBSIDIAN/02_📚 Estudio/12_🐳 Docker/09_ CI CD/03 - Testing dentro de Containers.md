---
tags:
  - Docker
  - DevOps
  - Docker-CICD
Creado: 2026-07-28
Relacionado:
  - CI CD
  - DOCKER
---

# 03 - Testing dentro de Containers

## Que vas a aprender

En este capitulo vas a aprender a estructurar la **ejecucion de tests dentro de containers** en un pipeline de CI: usar una etapa de multi-stage build dedicada a testing, y levantar dependencias (base de datos, Redis) con Compose durante el pipeline para tests de integracion.

## Explicacion

Correr tests dentro de un container (en vez de directamente en el agente de CI) garantiza que los tests corran exactamente en el mismo entorno que la imagen de produccion: mismas versiones de sistema operativo, librerias del sistema, y runtime. Esto se puede estructurar como una **etapa adicional del multi-stage build** (ver capitulo de Imagenes) dedicada exclusivamente a correr la suite de tests, que no se copia a la imagen final pero cuyo fallo detiene el build completo.

Para tests que necesitan dependencias reales (base de datos, cache), el patron estandar en CI es levantar esas dependencias con Docker Compose como parte del pipeline, correr los tests contra ellas, y despues destruir todo el entorno efimero, sin dejar estado persistente entre ejecuciones distintas del pipeline.

## Modelo mental

Pensa correr tests dentro de un container de CI como hacer un ensayo general en el mismo escenario exacto donde se presentara la funcion real (produccion), en vez de ensayar en un salon distinto que se parece pero no es identico. Cualquier diferencia sutil de entorno que pudiera causar un bug queda capturada durante el ensayo, no durante la funcion real.

## Ejemplo base

```dockerfile
# Etapa de test dentro del multi-stage build
FROM node:20-alpine AS test
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run test        # si falla, el build completo falla aca

FROM node:20-alpine AS builder
# ... build normal, sin depender de la etapa de test
```

```yaml
# docker-compose.ci.yml: dependencias efimeras para tests de integracion en CI
services:
  db-test:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: test
  tests:
    build:
      context: .
      target: test
    environment:
      DB_HOST: db-test
    depends_on:
      db-test: { condition: service_healthy }
```

## Como se usa en un proyecto real

1. Se agrega una etapa `test` en el multi-stage build cuyo fallo detiene el pipeline antes de construir o publicar la imagen final.
2. Se levantan dependencias reales (base de datos, cache) via Compose especificamente para el pipeline de CI, con datos efimeros que se destruyen al terminar.
3. Se ejecutan tests unitarios directamente dentro del contexto de build (sin necesitar dependencias externas), y tests de integracion/e2e contra el stack levantado con Compose.
4. Se paralelizan tests independientes (unitarios vs e2e) en jobs distintos del pipeline cuando el tiempo total lo justifica.
5. Se limpia siempre el entorno de Compose al finalizar el pipeline (`docker compose down -v`), sin importar si los tests pasaron o fallaron.

## Criterio junior

Un junior debe poder agregar una etapa de test a un multi-stage build que detenga el pipeline si los tests fallan.

## Criterio mid

Un mid debe levantar dependencias reales con Compose en el pipeline de CI para tests de integracion, y asegurar la limpieza correcta del entorno efimero al finalizar.

## Criterio senior

Un senior debe diseñar la estrategia completa de testing en CI (que corre en la etapa de build vs en un job separado, paralelizacion, tiempo total del pipeline) balanceando cobertura real contra velocidad de feedback para el equipo.

## Errores comunes

- Correr tests fuera del container, en el entorno del agente de CI directamente, perdiendo la garantia de que corren en el mismo entorno que produccion.
- No limpiar el entorno de Compose al finalizar el pipeline, dejando recursos huerfanos en el agente de CI.
- No paralelizar tests independientes cuando el tiempo total del pipeline se vuelve un cuello de botella para el equipo.
- Dejar que un fallo de test no detenga el pipeline correctamente, permitiendo que una imagen con tests fallidos se publique de todas formas.

## Practica

- [ ] Agrega una etapa de test a un multi-stage build de un proyecto de practica.
- [ ] Verifica que un test fallido efectivamente detiene el build completo.
- [ ] Levanta dependencias con Compose en un pipeline de CI y corre tests de integracion contra ellas.
- [ ] Verifica que el entorno de Compose se limpia correctamente al finalizar, sin importar el resultado de los tests.
- [ ] Evalua si tu suite de tests se beneficiaria de paralelizacion en jobs distintos del pipeline.

## Preguntas de repaso

- Que ventaja da correr tests dentro de un container sobre correrlos directamente en el agente de CI?
- Como estructurarias una etapa de test dentro de un multi-stage build para que su fallo detenga el pipeline?
- Por que es importante limpiar el entorno de Compose al finalizar el pipeline, sin importar el resultado?
- Cuando se justifica paralelizar tests en jobs distintos del pipeline?
