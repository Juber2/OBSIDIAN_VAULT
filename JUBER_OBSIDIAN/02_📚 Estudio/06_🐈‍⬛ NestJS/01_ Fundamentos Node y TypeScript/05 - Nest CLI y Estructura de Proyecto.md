---
tags:
  - NestJS
  - TypeScript
  - Nest-Fundamentos
Creado: 2026-07-27
Relacionado:
  - Fundamentos Node y TypeScript
  - NESTJS
---

# 05 - Nest CLI y Estructura de Proyecto

## Que vas a aprender

En este capitulo vas a dominar el **Nest CLI**: generacion de recursos (modulos, controllers, servicios), la estructura de carpetas por defecto, y como organizar un proyecto Nest real por dominio en vez de por tipo de archivo.

## Explicacion

El Nest CLI (`nest new`, `nest generate`) automatiza la creacion de modulos, controllers, servicios, y los registra automaticamente en el modulo correspondiente. Esto ahorra boilerplate pero tambien puede generar una estructura plana que no escala si se usa sin criterio.

La estructura por defecto agrupa por tipo (`controllers/`, `services/`) en proyectos chicos, pero en proyectos reales conviene organizar **por dominio/feature** (`users/`, `orders/`, `payments/`), donde cada carpeta tiene su propio modulo, controller, servicio, DTOs y entidades. Esto se alinea con como despues se piensa arquitectura modular y microservicios.

## Modelo mental

Pensa cada carpeta de feature como un "mini-modulo Nest" autocontenido: si mañana ese feature se separa en su propio microservicio, deberia poder moverse casi sin tocar el resto del proyecto.

## Ejemplo base

```bash
nest generate module users
nest generate controller users
nest generate service users
```

```
src/
  users/
    dto/
      create-user.dto.ts
    entities/
      user.entity.ts
    users.controller.ts
    users.service.ts
    users.module.ts
  app.module.ts
  main.ts
```

## Como se usa en un proyecto real

1. Se define desde el inicio si la organizacion sera por dominio (recomendado a partir de cierto tamaño) o por tipo (solo para proyectos muy chicos).
2. Se usa el CLI para generar recursos y mantener consistencia de nombres.
3. Cada modulo de dominio expone solo lo necesario (`exports`) y mantiene privado el resto.
4. El `AppModule` raiz solo importa modulos de dominio, sin logica propia.
5. Se documenta la convencion de estructura en el README del proyecto para nuevos integrantes.

## Criterio junior

Un junior debe poder usar el CLI para generar modulos, controllers y servicios, y entender la estructura de carpetas por defecto.

## Criterio mid

Un mid debe poder reorganizar un proyecto de "por tipo" a "por dominio" sin romper imports, y justificar la eleccion de estructura segun el tamaño del proyecto.

## Criterio senior

Un senior debe definir la convencion de estructura para el equipo, revisar que los modulos no se acoplen entre si de forma implicita, y anticipar como esa estructura facilita o dificulta una futura separacion en microservicios.

## Errores comunes

- Dejar todo en una carpeta plana `controllers/`, `services/` en un proyecto grande, perdiendo cohesion por dominio.
- Exportar todo desde cada modulo "por si acaso", generando acoplamiento oculto.
- No usar el CLI y escribir boilerplate a mano de forma inconsistente entre desarrolladores.
- Mezclar logica de infraestructura (DB, config) dentro de modulos de dominio sin separacion clara.

## Practica

- [ ] Genera un modulo, controller y servicio nuevo con el CLI.
- [ ] Reorganiza un proyecto de ejemplo de estructura por tipo a estructura por dominio.
- [ ] Define que exporta y que no exporta cada modulo.
- [ ] Documenta la convencion de carpetas en un README.
- [ ] Anota que feature separarias primero si tuvieras que migrar a microservicios.

## Preguntas de repaso

- Que genera automaticamente `nest generate module`?
- Por que conviene organizar por dominio en proyectos grandes?
- Que deberia exportar un modulo de dominio y que deberia quedar privado?
- Como facilita una buena estructura una futura migracion a microservicios?
