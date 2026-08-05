---
tags:
  - dotnet
  - laboratorio
  - devops
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos dotNET
  - NET
---

# Laboratorio 07 - Docker observabilidad y deploy

## Objetivo

Preparar la API para correr fuera de tu maquina: Docker, health checks, logs estructurados, traces y pipeline básico.

## Entregables

- Dockerfile multi-stage.
- docker-compose con API y DB.
- Health endpoint.
- Logging con correlation id.
- Pipeline que corre build y test.

## Observabilidad minima

Cada request debe poder rastrearse. Registra errores con contexto, no con datos sensibles.

## Deploy

Puede ser local, Azure App Service o container. Lo importante es documentar variables, secretos y rollback.

## Errores comunes

- Secrets en repo
- Logs con tokens
- Deploy sin health check

## Practica

- [ ] Crear Dockerfile
- [ ] Levantar con compose
- [ ] Documentar variables de entorno

## Relacionado

- [[Laboratorios Practicos dotNET]]
- [[NET]]
