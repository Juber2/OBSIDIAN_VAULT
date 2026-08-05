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

# 01 - Build de Imagenes en Pipelines

## Que vas a aprender

En este capitulo vas a aprender a estructurar el paso de **build de imagen Docker dentro de un pipeline de CI**: como se ejecuta Docker dentro de un agente de CI (Docker-in-Docker vs socket montado), y las consideraciones basicas de ese entorno.

## Explicacion

Correr `docker build` dentro de un pipeline de CI requiere que el agente de CI tenga acceso a un daemon Docker. Hay dos enfoques principales: **Docker-in-Docker (DinD)**, donde se corre un daemon Docker completo dentro del container del agente de CI (mas aislado, pero con overhead y algunas limitaciones de compatibilidad), o **montar el socket del daemon del host** (`/var/run/docker.sock`), donde el agente de CI usa el mismo daemon que corre en la maquina host (mas simple y rapido, pero con implicancias de seguridad porque el agente obtiene efectivamente control total sobre Docker del host).

La mayoria de plataformas de CI modernas (GitHub Actions, GitLab CI) ofrecen runners con Docker ya preconfigurado, abstrayendo buena parte de esta complejidad, pero entender que ocurre por debajo ayuda a diagnosticar problemas y tomar decisiones de seguridad informadas.

## Modelo mental

Pensa Docker-in-Docker como construir una casa de muñecas completa (un daemon Docker independiente) dentro de la casa real, aislada pero con recursos duplicados. Montar el socket del host es como darle al constructor de la casa de muñecas las llaves de la casa real completa para que use las herramientas ya existentes ahi, mas rapido pero con mucho mas poder del que estrictamente necesita.

## Ejemplo base

```yaml
# GitHub Actions: Docker ya disponible en runners estandar (aborda la complejidad por vos)
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build image
        run: docker build -t mi-imagen:${{ github.sha }} .
```

```yaml
# GitLab CI: ejemplo de servicio Docker-in-Docker explicito
build:
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t mi-imagen:$CI_COMMIT_SHA .
```

## Como se usa en un proyecto real

1. Se usan los runners preconfigurados con Docker de la plataforma de CI (GitHub Actions, GitLab CI) siempre que sea posible, en vez de configurar Docker-in-Docker manualmente sin necesidad.
2. Se evalua el tradeoff de seguridad de montar el socket del host solo cuando el rendimiento es critico y el entorno de CI es confiable (no compartido con codigo de terceros no confiable).
3. Se cachea el build (ver capitulo de Optimizacion de Imagenes) especificamente configurado para el contexto de CI, dado que cada ejecucion puede correr en un agente efimero sin cache local persistente por defecto.
4. Se mide el tiempo del paso de build como parte del tiempo total del pipeline, identificando si es un cuello de botella que amerita optimizacion.
5. Se documenta claramente en la configuracion de CI que enfoque (DinD o socket montado) se usa y por que, para futuros mantenedores del pipeline.

## Criterio junior

Un junior debe poder configurar un paso basico de build de imagen en un pipeline de CI usando la integracion estandar de la plataforma.

## Criterio mid

Un mid debe entender la diferencia entre Docker-in-Docker y socket montado, y elegir el enfoque apropiado segun las restricciones de seguridad del entorno de CI.

## Criterio senior

Un senior debe evaluar los riesgos de seguridad de cada enfoque para el contexto especifico de la organizacion, y optimizar el tiempo total del pipeline de build como parte de la productividad general del equipo.

## Errores comunes

- Montar el socket del daemon del host en un entorno de CI compartido con codigo no confiable, otorgando control excesivo sobre el host.
- No aprovechar cache entre ejecuciones de CI, repitiendo builds lentos desde cero en cada pipeline sin necesidad.
- Configurar Docker-in-Docker manualmente cuando la plataforma de CI ya ofrece una integracion estandar mas simple.
- No medir el tiempo del paso de build, perdiendo visibilidad de si es un cuello de botella real del pipeline.

## Practica

- [ ] Configura un paso de build de imagen en un pipeline de CI real o de practica.
- [ ] Investiga si tu plataforma de CI usa Docker-in-Docker o socket montado por defecto.
- [ ] Mide el tiempo que toma el paso de build dentro del pipeline completo.
- [ ] Evalua el riesgo de seguridad de montar el socket del host en tu entorno de CI especifico.
- [ ] Documenta el enfoque elegido y su justificacion en la configuracion del pipeline.

## Preguntas de repaso

- Que diferencia hay entre Docker-in-Docker y montar el socket del daemon del host?
- Que riesgo de seguridad especifico tiene montar el socket del host en un entorno de CI compartido?
- Por que la mayoria de plataformas modernas de CI abstraen esta complejidad con runners preconfigurados?
- Como medirias si el paso de build de imagen es un cuello de botella en tu pipeline?
