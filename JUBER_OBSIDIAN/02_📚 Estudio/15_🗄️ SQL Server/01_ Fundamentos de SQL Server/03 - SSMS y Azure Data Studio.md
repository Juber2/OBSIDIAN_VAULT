---
tags:
  - SQLServer
  - Bases-de-Datos
  - SQLServer-Fundamentos
Creado: 2026-07-29
Relacionado:
  - Fundamentos de SQL Server
  - SQL SERVER
---

# 03 - SSMS y Azure Data Studio

## Que vas a aprender

En este capitulo vas a aprender las herramientas cliente estandar de SQL Server: **SQL Server Management Studio (SSMS)**, la herramienta tradicional completa de Windows, y **Azure Data Studio**, la alternativa moderna multiplataforma.

## Explicacion

**SSMS** es la herramienta administrativa completa tradicional de SQL Server: interfaz grafica rica para gestionar bases de datos, escribir T-SQL, ver planes de ejecucion visualmente (ver capitulo de Indices), configurar backups, jobs, seguridad, y practicamente cualquier tarea administrativa, pero **solo disponible en Windows**. **Azure Data Studio** es una alternativa mas moderna y liviana, multiplataforma (Windows, Mac, Linux, relevante si trabajas en el WSL2 ya visto en el libro de Linux), con una experiencia mas cercana a VS Code (extensiones, temas, notebooks de SQL), aunque con menos funcionalidad administrativa completa que SSMS.

Para trabajo diario de desarrollo (escribir queries, ver resultados, explorar esquemas), Azure Data Studio es suficiente y mas agil; para tareas administrativas profundas (configuracion de Always On, perfiles de seguridad complejos, ver capitulos de Alta Disponibilidad y Administracion), SSMS sigue siendo mas completo. Ambas herramientas se conectan al mismo motor SQL Server sin importar donde corra (local, Docker, Azure).

## Modelo mental

Pensa SSMS como una caja de herramientas completa de un tecnico especializado, con cada herramienta especifica para cada tarea administrativa imaginable, pero pesada y solo disponible en un taller especifico (Windows). Azure Data Studio es como una caja de herramientas mas liviana y portable que llevas a cualquier lugar (multiplataforma), suficiente para el 90% del trabajo diario, aunque para reparaciones muy especializadas todavia necesitas volver al taller completo.

## Ejemplo base

```
Comparativa practica:

| Tarea                              | SSMS | Azure Data Studio |
|--------------------------------------|--------|-----------------------|
| Escribir y ejecutar queries          | Si     | Si                    |
| Ver planes de ejecucion visualmente  | Si     | Si (con extension)    |
| Configurar Always On                 | Si     | Limitado              |
| Notebooks de SQL con markdown        | No     | Si                    |
| Disponibilidad                       | Windows| Multiplataforma       |
```

## Como se usa en un proyecto real

1. Se usa Azure Data Studio para el trabajo diario de desarrollo, especialmente si el equipo trabaja en entornos multiplataforma o WSL2 (ver libro de Linux).
2. Se usa SSMS para tareas administrativas profundas que requieren su interfaz completa (configuracion de Always On, perfiles de seguridad avanzados).
3. Se aprovechan los notebooks de SQL de Azure Data Studio para documentar procedimientos de administracion de forma ejecutable y explicada, similar en espiritu a los notebooks de Jupyter.
4. Se conecta cualquiera de las dos herramientas al mismo SQL Server sin importar donde corra, incluyendo instancias en Docker (ver capitulo correspondiente) o Azure SQL Database.
5. Se documenta que herramienta usa cada integrante del equipo segun su sistema operativo y tipo de tarea habitual.

## Criterio junior

Un junior debe poder conectarse a una instancia de SQL Server con Azure Data Studio o SSMS y ejecutar queries basicas.

## Criterio mid

Un mid debe usar ambas herramientas segun la tarea (SSMS para administracion profunda, Azure Data Studio para desarrollo agil).

## Criterio senior

Un senior debe estandarizar que herramientas usa el equipo segun su contexto de trabajo, documentando el proceso de conexion a distintos ambientes (local, Docker, Azure).

## Errores comunes

- Depender exclusivamente de SSMS cuando el equipo trabaja en entornos multiplataforma, sin considerar Azure Data Studio como alternativa mas agil.
- No aprovechar los notebooks de SQL de Azure Data Studio para documentar procedimientos administrativos de forma ejecutable.
- No conocer que tareas administrativas profundas todavia requieren SSMS, quedando limitado al intentar todo con Azure Data Studio.
- No estandarizar herramientas en el equipo, generando fragmentacion de experiencia entre distintos integrantes.

## Practica

- [ ] Instala Azure Data Studio y conectate a una instancia de SQL Server de practica.
- [ ] Ejecuta las mismas queries basicas en SSMS (si tenes acceso a Windows) y compara la experiencia.
- [ ] Explora un plan de ejecucion visual en cualquiera de las dos herramientas.
- [ ] Crea un notebook de SQL simple en Azure Data Studio documentando un procedimiento.
- [ ] Documenta que herramienta recomendarias para cada tipo de tarea en tu equipo.

## Preguntas de repaso

- Que ventaja de portabilidad da Azure Data Studio sobre SSMS?
- Que tareas administrativas profundas todavia requieren tipicamente SSMS?
- Para que sirven los notebooks de SQL de Azure Data Studio?
- Como conectarias cualquiera de estas herramientas a una instancia de SQL Server corriendo en Docker?
