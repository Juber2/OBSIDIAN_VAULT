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

# 02 - Cache entre Builds de CI

## Que vas a aprender

En este capitulo vas a aprender a resolver el problema especifico de cache en CI: los **agentes efimeros** (que empiezan sin ningun cache local) y como usar cache remoto para que cada build de CI no empiece siempre desde cero.

## Explicacion

A diferencia de una laptop de desarrollo, donde el cache de capas de Docker persiste entre builds sucesivos, muchos entornos de CI usan **agentes efimeros**: cada ejecucion del pipeline arranca en una maquina virtual o container nuevo, sin ningun cache de Docker previo disponible localmente. Sin abordar esto especificamente, cada build de CI reinstala dependencias desde cero, incluso si nada cambio desde el build anterior.

La solucion es usar **cache remoto** (`--cache-from`/`--cache-to` de BuildKit, ya visto en el capitulo de Optimizacion), tipicamente apuntando a un registry: el pipeline sube el cache generado al final de cada build, y lo descarga al inicio del siguiente, simulando la persistencia que existiria en una maquina de desarrollo fija. Muchas plataformas de CI (GitHub Actions con su cache nativo, GitLab CI) tambien ofrecen mecanismos propios de cache que pueden combinarse con el cache de Docker.

## Modelo mental

Pensa un agente de CI efimero como un cocinero nuevo que llega cada dia a una cocina completamente vacia, sin ningun ingrediente pre-cortado del dia anterior. El cache remoto es como dejarle al cocinero de mañana una nota con los ingredientes ya pre-cortados guardados en un lugar accesible (el registry), para que no tenga que empezar literalmente de cero cada vez.

## Ejemplo base

```yaml
# GitHub Actions con cache nativo de Docker Buildx
- uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: mi-registro/api:${{ github.sha }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

```bash
# Equivalente generico con cache en un registry
docker buildx build \
  --cache-from type=registry,ref=mi-registro/api:cache \
  --cache-to type=registry,ref=mi-registro/api:cache,mode=max \
  -t mi-registro/api:$COMMIT_SHA --push .
```

## Como se usa en un proyecto real

1. Se configura cache remoto (via registry o el cache nativo de la plataforma de CI) desde el inicio del pipeline, dado que el impacto en velocidad suele ser significativo.
2. Se usa `mode=max` cuando se quiere cachear todas las capas intermedias, no solo las de la imagen final, maximizando el reuso entre builds distintos que comparten pasos.
3. Se mide el tiempo de build con y sin cache remoto activo, para verificar el beneficio real obtenido en el contexto especifico del proyecto.
4. Se combina con la estrategia de orden de instrucciones del Dockerfile (ver capitulo de Imagenes) para maximizar los hits de cache real entre commits sucesivos.
5. Se limpia periodicamente el cache remoto si crece de forma descontrolada, aplicando la misma logica de retencion vista para imagenes normales.

## Criterio junior

Un junior debe entender por que los agentes de CI efimeros pierden el cache local entre ejecuciones, y poder configurar un cache remoto basico.

## Criterio mid

Un mid debe medir el impacto real del cache remoto en los tiempos de pipeline, y ajustar la configuracion (`mode=max` vs default) segun las necesidades del proyecto.

## Criterio senior

Un senior debe optimizar la estrategia de cache de CI de todo el equipo, balanceando velocidad de pipeline contra el costo de almacenamiento del cache remoto, y estableciendolo como estandar de las plantillas de CI del equipo.

## Errores comunes

- No configurar ningun cache remoto en CI, dejando que cada build reinstale dependencias desde cero innecesariamente en cada ejecucion.
- No medir el impacto real del cache configurado, asumiendo una mejora que en la practica no es significativa para ese proyecto especifico.
- Dejar crecer el cache remoto indefinidamente sin ninguna politica de limpieza.
- No combinar el cache remoto con un buen ordenamiento de instrucciones del Dockerfile, perdiendo buena parte del beneficio potencial.

## Practica

- [ ] Configura cache remoto (via registry o cache nativo de tu plataforma de CI) para un pipeline de build.
- [ ] Mide el tiempo de build en CI antes y despues de activar el cache remoto.
- [ ] Prueba la diferencia entre el modo de cache default y `mode=max`.
- [ ] Verifica que el cache remoto realmente se reusa entre dos ejecuciones sucesivas del pipeline con cambios menores.
- [ ] Documenta la configuracion de cache de CI como parte del estandar de pipelines de tu equipo.

## Preguntas de repaso

- Por que un agente de CI efimero pierde el cache local de Docker entre ejecuciones sucesivas?
- Que resuelve el cache remoto (`--cache-from`/`--cache-to` apuntando a un registry) en este contexto?
- Que diferencia hay entre el modo de cache default y `mode=max`?
- Como medirias objetivamente el impacto real del cache remoto en tu pipeline?
