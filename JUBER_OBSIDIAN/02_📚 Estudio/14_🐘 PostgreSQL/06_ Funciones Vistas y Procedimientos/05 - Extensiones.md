---
tags:
  - PostgreSQL
  - Funciones
  - Postgres-Funciones
Creado: 2026-07-29
Relacionado:
  - Funciones Vistas y Procedimientos
  - POSTGRESQL
---

# 05 - Extensiones

## Que vas a aprender

En este capitulo vas a cerrar el capitulo aprendiendo el sistema de **extensiones** de Postgres: como instalarlas, y las mas usadas en proyectos reales: `pg_stat_statements` (analisis de queries), `uuid-ossp`/`pgcrypto` (generacion de UUIDs), y `pg_trgm` (busqueda de texto por similitud).

## Explicacion

Una extension es un modulo empaquetado que agrega funcionalidad completa al motor sin necesitar recompilar Postgres ni salir del ecosistema del motor: nuevas funciones, tipos de datos, o incluso operadores. Se instalan con `CREATE EXTENSION nombre` dentro de una base de datos especifica (no globalmente para todo el cluster). Este sistema es lo que explica que Postgres pueda hacer cosas tan distintas como analisis geoespacial (PostGIS) o busqueda vectorial para embeddings de IA (pgvector) sin ser parte del nucleo del motor.

Extensiones especialmente utiles en el dia a dia: **`pg_stat_statements`** registra estadisticas de todas las queries ejecutadas (tiempo total, numero de ejecuciones), siendo la herramienta fundamental de monitoreo de performance en produccion (ver capitulo de Performance Avanzada). **`pgcrypto`** provee `gen_random_uuid()` (usado ya en el capitulo de Modelado) y funciones de cifrado. **`pg_trgm`** permite busqueda de texto por similitud (util para "buscar aunque haya errores de tipeo"), indexable con GIN (ver capitulo de Indices).

## Modelo mental

Pensa el sistema de extensiones como aplicaciones instalables en un smartphone: el sistema operativo base (Postgres) ya es completo y funcional, pero podes instalar apps especializadas (extensiones) que agregan capacidades especificas (GPS avanzado, edicion de fotos profesional) sin necesitar cambiar el sistema operativo en si.

## Ejemplo base

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS pg_trgm;

\dx    -- lista extensiones instaladas en la base actual

-- pg_stat_statements: top queries por tiempo total acumulado
SELECT query, calls, total_exec_time, mean_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC LIMIT 10;

-- pg_trgm: busqueda por similitud, tolerante a errores de tipeo
SELECT nombre, similarity(nombre, 'jhon') AS score
FROM usuarios
WHERE nombre % 'jhon'    -- operador de similitud
ORDER BY score DESC;
```

## Como se usa en un proyecto real

1. Se instala `pg_stat_statements` en todo ambiente de produccion desde el inicio, dado que es la base de cualquier trabajo serio de optimizacion de performance (ver capitulo siguiente).
2. Se usa `pgcrypto` para generar UUIDs con `gen_random_uuid()`, ya usado en el capitulo de Modelado como alternativa de clave primaria.
3. Se evalua `pg_trgm` para funcionalidades de busqueda tolerante a errores tipograficos, comun en buscadores de productos o usuarios.
4. Se verifica que extensiones necesarias estan disponibles antes de comprometerse con un proveedor cloud especifico (ver capitulo de Postgres en la Nube), dado que algunos servicios gestionados restringen que extensiones se pueden instalar.
5. Se documenta que extensiones usa cada proyecto y por que, como parte de los requisitos de infraestructura necesarios para reproducir el entorno.

## Criterio junior

Un junior debe poder instalar una extension basica con `CREATE EXTENSION` y listar extensiones instaladas con `\dx`.

## Criterio mid

Un mid debe usar `pg_stat_statements` para identificar las queries mas costosas de un sistema, y aplicar `pg_trgm` para busqueda por similitud cuando corresponda.

## Criterio senior

Un senior debe estandarizar que extensiones son parte del stack base de todo proyecto del equipo, y verificar su disponibilidad antes de comprometerse con proveedores cloud especificos.

## Errores comunes

- No instalar `pg_stat_statements` desde el inicio, perdiendo visibilidad historica de queries costosas hasta que ya es un problema evidente en produccion.
- Asumir que una extension necesaria estara disponible en cualquier proveedor cloud sin verificarlo antes de comprometerse con esa plataforma.
- No usar `pg_trgm` cuando una funcionalidad de busqueda realmente lo necesitaria, implementando busqueda tolerante a errores de forma mas compleja del lado de la aplicacion.
- No documentar las extensiones requeridas por un proyecto, dificultando reproducir el entorno en un ambiente nuevo.

## Practica

- [ ] Instala `pg_stat_statements`, `pgcrypto` y `pg_trgm` en una base de practica.
- [ ] Lista las extensiones instaladas con `\dx`.
- [ ] Consulta `pg_stat_statements` para ver las queries mas costosas ejecutadas hasta ahora.
- [ ] Prueba una busqueda por similitud con `pg_trgm` sobre datos con errores de tipeo intencionales.
- [ ] Verifica que extensiones estan disponibles en un proveedor cloud que consideres usar.

## Preguntas de repaso

- Que es una extension de Postgres y como se instala?
- Por que `pg_stat_statements` es fundamental para trabajo de optimizacion de performance?
- Que problema resuelve `pg_trgm` que una busqueda de texto exacta no resuelve?
- Que verificarias antes de comprometerte con un proveedor cloud respecto a extensiones necesarias?
