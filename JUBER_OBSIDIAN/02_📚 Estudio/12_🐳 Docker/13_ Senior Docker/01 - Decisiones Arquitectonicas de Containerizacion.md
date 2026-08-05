---
tags:
  - Docker
  - DevOps
  - Docker-Senior
Creado: 2026-07-28
Relacionado:
  - Senior Docker
  - DOCKER
---

# 01 - Decisiones Arquitectonicas de Containerizacion

## Que vas a aprender

En este capitulo vas a desarrollar el criterio senior para decidir **cuando containerizar y cuando no**, y como estructurar containers alrededor de limites de servicio (un proceso por container) en vez de amontonar todo junto.

## Explicacion

No todo se beneficia de containerizarse: un script simple que corre una vez al mes puede no justificar el overhead de mantener un Dockerfile y una imagen. La pregunta senior, igual que con cualquier decision arquitectonica (ver el capitulo Senior del libro de NestJS), es que problema real resuelve la containerizacion en cada caso especifico: portabilidad entre entornos, aislamiento de dependencias, facilidad de escalado horizontal.

Un principio de diseño central es **un proceso principal por container**: en vez de meter una aplicacion, su base de datos, y un cron job todos en el mismo container (un anti-patron comun en quienes recien empiezan), cada responsabilidad va en su propio container, comunicandose por red (capitulo de Redes) y orquestados juntos con Compose o un orquestador real. Esto respeta el mismo principio de separacion de responsabilidades visto en arquitectura de software en general.

## Modelo mental

Pensa cada container como un empleado especializado en una unica tarea, en vez de una persona que intenta hacer de recepcionista, contador y tecnico de soporte al mismo tiempo: cada especialista es mas facil de reemplazar, escalar independientemente, y diagnosticar cuando algo falla especificamente en su area.

## Ejemplo base

```
Anti-patron: un unico container "todo en uno"
CMD service postgresql start && service redis start && node dist/main.js

Patron correcto: servicios separados, cada uno con su responsabilidad
services:
  api: { build: . }
  db: { image: postgres:16 }
  redis: { image: redis:7-alpine }
```

## Como se usa en un proyecto real

1. Se evalua para cada componente si containerizarlo realmente resuelve un problema (portabilidad, aislamiento, escalado) antes de hacerlo por defecto.
2. Se diseña cada container alrededor de un unico proceso principal, delegando la orquestacion de multiples procesos relacionados a Compose o al orquestador, no a scripts dentro de un unico container.
3. Se documenta la arquitectura de containers de un sistema (que container hace que, como se comunican) como parte de la documentacion tecnica del proyecto (ver capitulo de Documentacion en el libro de NestJS).
4. Se revisa en code review cualquier Dockerfile que intente correr multiples procesos no relacionados dentro del mismo container.
5. Se considera el ciclo de vida independiente de cada componente: si dos cosas necesitan escalar, actualizarse o reiniciarse en momentos distintos, son buenos candidatos a containers separados.

## Criterio junior

Un junior debe entender el principio de un proceso principal por container y reconocer un anti-patron de "todo en uno" cuando lo ve.

## Criterio mid

Un mid debe diseñar la separacion de containers de un sistema nuevo respetando limites de responsabilidad claros entre servicios.

## Criterio senior

Un senior debe decidir con criterio cuando containerizar un componente aporta valor real, diseñar la arquitectura completa de containers de un sistema, y establecer esto como estandar revisado en code review para todo el equipo.

## Errores comunes

- Meter multiples procesos no relacionados en un unico container "para simplificar", perdiendo la capacidad de escalar y actualizar cada uno independientemente.
- Containerizar componentes que no se benefician realmente de ello, agregando complejidad de mantenimiento sin beneficio proporcional.
- No documentar la arquitectura de containers de un sistema, dificultando el onboarding y la toma de decisiones futuras.
- No considerar el ciclo de vida independiente de cada componente al decidir los limites entre containers.

## Practica

- [ ] Identifica un container "todo en uno" (real o hipotetico) y rediseñalo respetando el principio de un proceso por container.
- [ ] Evalua un componente de tu sistema que no esta containerizado y decide si deberia estarlo, con justificacion.
- [ ] Documenta la arquitectura de containers de un proyecto real que conozcas.
- [ ] Define una regla de code review sobre "un proceso principal por container" para tu equipo.
- [ ] Anota un caso donde NO valdria la pena containerizar algo.

## Preguntas de repaso

- Por que el principio de "un proceso principal por container" facilita escalado y mantenimiento independiente?
- Que pregunta senior deberias hacerte antes de containerizar cualquier componente?
- Que señal indica que dos componentes deberian estar en containers separados en vez de juntos?
- Que caso justificaria NO containerizar un componente?
