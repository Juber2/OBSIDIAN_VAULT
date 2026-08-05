---
tags:
  - NestJS
  - TypeScript
  - Nest-DevOps
Creado: 2026-07-27
Relacionado:
  - DevOps Deploy
  - NESTJS
---

# 03 - CI CD NestJS

## Que vas a aprender

En este capitulo vas a aprender a diseñar un pipeline de **CI/CD** para un proyecto NestJS: etapas de lint, test, build, y despliegue automatizado, con las practicas que evitan que un pipeline verde termine en un deploy roto.

## Explicacion

Un pipeline de CI tipico para Nest corre, en orden, y fallando rapido ante el primer error: instalacion de dependencias con lockfile exacto (`npm ci`), lint, tests unitarios, tests e2e (contra una base de datos efimera del pipeline), build de produccion, y build/push de la imagen Docker. Solo si todas esas etapas pasan, el pipeline de CD toma el control para desplegar.

La clave de un buen pipeline es que sea **reproducible** (el mismo commit siempre produce el mismo resultado) y que corra las migrations de base de datos como parte del deploy, no manualmente, para evitar drift entre el codigo desplegado y el esquema de base de datos.

## Modelo mental

Pensa el pipeline de CI/CD como una serie de controles de calidad en una linea de produccion: cada estacion verifica algo especifico, y si algo no pasa el control, el producto (el deploy) no avanza a la siguiente estacion, evitando que algo defectuoso llegue al cliente final.

## Ejemplo base

```yaml
# .github/workflows/ci.yml (fragmento conceptual)
jobs:
  test:
    steps:
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run test:e2e
      - run: npm run build

  deploy:
    needs: test
    steps:
      - run: docker build -t app:${{ github.sha }} .
      - run: docker push app:${{ github.sha }}
      - run: npm run migration:run # antes de activar la nueva version
```

## Como se usa en un proyecto real

1. El pipeline falla rapido: lint antes que tests, tests antes que build, evitando gastar tiempo de CI en pasos costosos si algo basico ya esta mal.
2. Los tests e2e corren contra una base de datos efimera creada especificamente para el pipeline, nunca contra una compartida.
3. Las migrations se ejecutan como parte del proceso de deploy, de forma automatizada y versionada, nunca manualmente por un desarrollador conectandose a produccion.
4. Se implementa deploy con estrategia de bajo riesgo (blue-green, rolling update, canary) segun la criticidad del sistema.
5. Se mantiene un mecanismo de rollback rapido si el nuevo deploy presenta problemas detectados post-despliegue.

## Criterio junior

Un junior debe entender las etapas basicas de un pipeline de CI (lint, test, build) y por que se ejecutan en ese orden.

## Criterio mid

Un mid debe configurar un pipeline completo con tests e2e contra base de datos efimera, y automatizar la ejecucion de migrations como parte del deploy.

## Criterio senior

Un senior debe diseñar la estrategia de deploy de bajo riesgo del sistema (blue-green, canary), definir el mecanismo de rollback, y balancear velocidad del pipeline con cobertura de verificacion antes de cada deploy a produccion.

## Errores comunes

- Ejecutar migrations manualmente en produccion en vez de automatizarlas dentro del pipeline de deploy.
- No correr tests e2e en CI, dejando que bugs de integracion lleguen a produccion pese a tener tests escritos.
- No tener un mecanismo de rollback rapido, quedando expuesto ante un deploy problematico sin forma agil de revertir.
- Compartir la misma base de datos entre el pipeline de CI y el ambiente de desarrollo, generando interferencia entre ejecuciones.

## Practica

- [ ] Diseña las etapas de un pipeline de CI para un proyecto Nest de practica.
- [ ] Configura tests e2e para correr contra una base de datos efimera en el pipeline.
- [ ] Automatiza la ejecucion de migrations como parte del proceso de deploy.
- [ ] Investiga una estrategia de deploy de bajo riesgo (blue-green o canary) aplicable a tu contexto.
- [ ] Anota el mecanismo de rollback que usarias ante un deploy problematico.

## Preguntas de repaso

- Por que el pipeline deberia fallar rapido, corriendo lint antes que tests pesados?
- Que riesgo tiene ejecutar migrations manualmente en produccion en vez de automatizarlas?
- Que ventaja da una base de datos efimera para los tests e2e del pipeline sobre una compartida?
- Que mecanismo de rollback tendrias listo ante un deploy que resulta problematico?
