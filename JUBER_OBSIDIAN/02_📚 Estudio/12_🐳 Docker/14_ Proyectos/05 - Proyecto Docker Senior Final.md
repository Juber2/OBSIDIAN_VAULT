---
tags:
  - Docker
  - DevOps
  - Docker-Proyectos
Creado: 2026-07-28
Relacionado:
  - Proyectos
  - DOCKER
---

# 05 - Proyecto Docker Senior Final

## Que vas a construir

El proyecto integrador del libro: un **stack full-stack completo production-ready**, tomando todo lo construido en los proyectos anteriores y llevandolo al nivel de los capitulos de Monitoreo, Docker para Diferentes Stacks, y Senior Docker.

## Alcance funcional

- Stack completo: frontend (Angular u otro), backend (NestJS/.NET), base de datos, cache, con la segmentacion de red del proyecto de Compose ya construido.
- Pipeline de CI/CD completo (del proyecto 3) integrado con este stack, publicando imagenes tageadas y trazables.
- Hardening aplicado (del proyecto 4) a cada imagen del stack.
- Logging estructurado con request ID de correlacion, health checks reales para cada servicio con estado.
- Limites de recursos (CPU/memoria) definidos para cada servicio, basados en una estimacion razonada de consumo.
- Documentacion completa: README de setup, y al menos dos ADRs de decisiones tomadas (por ejemplo: eleccion de imagen base, estrategia de deploy).

## Checklist tecnica

- [ ] Todo el stack levanta con un unico comando en una maquina limpia, sin pasos manuales adicionales.
- [ ] Cada servicio con estado tiene un health check real verificado, no trivial.
- [ ] Los logs de una misma request pueden correlacionarse entre servicios mediante un ID compartido.
- [ ] Cada imagen del stack corre como usuario no-root y esta razonablemente optimizada en tamaño.
- [ ] Existe una politica de retencion de imagenes documentada para el registry usado.
- [ ] Se documenta la estrategia de deploy elegida (rolling, blue-green, o justificacion de por que ninguna es necesaria todavia) con su razonamiento.

## Criterio de exito

El proyecto se considera completo cuando otra persona (o vos mismo, meses despues) puede clonar el repositorio, entender las decisiones tomadas leyendo los ADRs, levantar el stack completo con confianza, y diagnosticar un problema simulado usando el flujo sistematico de troubleshooting del capitulo Senior sin necesitar ayuda externa.

## Errores comunes a evitar

- Dejar el proyecto "funcionando en mi maquina" sin verificar que levanta igual en una maquina limpia desde cero.
- Aplicar optimizaciones y hardening sin medir ni documentar su impacto real.
- No conectar el pipeline de CI/CD con el stack completo, dejando el proceso de publicacion desconectado del resto del proyecto.
- Terminar el proyecto sin documentar el razonamiento detras de decisiones no obvias, perdiendo ese contexto con el tiempo.

## Cierre del libro

Completar este proyecto es la señal de haber recorrido el camino completo de este libro: desde entender que es un container hasta operar con criterio senior un sistema containerizado completo, seguro, observable y desplegable con confianza. El siguiente paso natural es aplicar exactamente este mismo criterio al [[NESTJS|backend NestJS]] o a StaffTrainingPlatform, cerrando el circulo entre ambos libros de este vault.
