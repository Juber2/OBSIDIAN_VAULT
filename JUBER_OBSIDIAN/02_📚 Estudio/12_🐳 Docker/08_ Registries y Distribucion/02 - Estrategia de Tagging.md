---
tags:
  - Docker
  - DevOps
  - Docker-Registries
Creado: 2026-07-28
Relacionado:
  - Registries y Distribucion
  - DOCKER
---

# 02 - Estrategia de Tagging

## Que vas a aprender

En este capitulo vas a aprender a diseñar una **estrategia de tagging** de imagenes coherente: por que evitar `latest` en produccion, versionado semantico, tags por hash de commit, y como combinar varios tags para el mismo build.

## Explicacion

El tag `latest` es el default cuando no se especifica ninguno, y es tentador usarlo siempre, pero es **ambiguo por definicion**: no dice nada sobre que version especifica del codigo contiene, y "el ultimo" cambia constantemente, haciendo imposible saber exactamente que esta corriendo en produccion o revertir con precision a una version anterior conocida.

La practica recomendada es tagear cada imagen con **multiples tags simultaneos** en el mismo build: uno inmutable y trazable (el hash corto del commit de Git, `sha-a1b2c3d`), y opcionalmente uno de version semantica (`1.4.0`) para releases formales, ademas de mantener `latest` solo como un puntero conveniente que se actualiza junto con el tag real, nunca como la unica referencia usada en despliegues.

## Modelo mental

Pensa `latest` como llamar a alguien simplemente "la persona mas joven de la familia": ese titulo cambia con el tiempo y no identifica a nadie de forma permanente. Un tag por hash de commit es como el documento de identidad de esa persona: siempre identifica exactamente a la misma persona, sin ambiguedad, sin importar cuanto tiempo pase.

## Ejemplo base

```bash
COMMIT_SHA=$(git rev-parse --short HEAD)

docker build -t mi-registro/api:sha-$COMMIT_SHA \
             -t mi-registro/api:1.4.0 \
             -t mi-registro/api:latest .

docker push mi-registro/api:sha-$COMMIT_SHA
docker push mi-registro/api:1.4.0
docker push mi-registro/api:latest
```

```yaml
# El deploy real SIEMPRE referencia el tag trazable, nunca latest
services:
  api:
    image: mi-registro/api:sha-a1b2c3d
```

## Como se usa en un proyecto real

1. Se tagea cada build con el hash corto del commit como tag primario e inmutable, usado en los deploys reales.
2. Se agrega version semantica (`major.minor.patch`) para releases formales dirigidos a usuarios o consumidores externos de la imagen.
3. `latest` se actualiza automaticamente en cada build de la rama principal, pero nunca es el tag referenciado directamente en configuraciones de deploy de produccion.
4. Se documenta la convencion de tagging del equipo, incluyendo que rama o evento dispara cada tipo de tag.
5. Se aprovecha la trazabilidad del tag por commit para poder revertir un deploy a una version exacta anterior sin ambiguedad.

## Criterio junior

Un junior debe entender por que `latest` es ambiguo y poder tagear una imagen con el hash de commit correspondiente.

## Criterio mid

Un mid debe implementar una estrategia de tagging con multiples tags simultaneos (commit, version semantica, latest) en el pipeline de CI/CD.

## Criterio senior

Un senior debe definir la convencion de tagging de toda la organizacion, asegurando que ningun despliegue de produccion dependa de `latest`, y que cualquier version desplegada sea trazable hasta el commit exacto que la genero.

## Errores comunes

- Desplegar en produccion referenciando `latest`, perdiendo la capacidad de saber exactamente que codigo esta corriendo o de revertir con precision.
- No tagear con el hash de commit, dificultando la trazabilidad entre una imagen desplegada y el codigo fuente exacto que la genero.
- Confundir version semantica con tag de build: no todo build de CI necesita un bump de version semantica formal.
- No documentar la convencion de tagging, generando inconsistencias entre distintos miembros del equipo.

## Practica

- [ ] Configura un pipeline que tagee cada build con el hash de commit y actualice `latest`.
- [ ] Despliega referenciando explicitamente el tag por commit, no `latest`.
- [ ] Simula un rollback aprovechando la trazabilidad del tag por commit a una version anterior conocida.
- [ ] Agrega tagging de version semantica para un release formal de ejemplo.
- [ ] Documenta la convencion de tagging de tu equipo.

## Preguntas de repaso

- Por que `latest` es ambiguo y poco confiable como referencia de despliegue en produccion?
- Que ventaja de trazabilidad da tagear con el hash corto del commit de Git?
- Cuando conviene agregar ademas un tag de version semantica al mismo build?
- Como aprovecharias un tagging trazable para hacer un rollback preciso ante un problema en produccion?
