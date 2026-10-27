---
tags:
  - Postulacion
  - PeriferiaIT
  - dotNET
  - CSharp
  - VB6
  - WinForms
  - Modernizacion
Creado: 2026-08-30
Estado: Por evaluar
Empresa: Periferia IT Group
Modalidad: Híbrido 2x3
Ubicacion: San Isidro, Lima
Contrato: Plazo fijo
Sueldo: Negociable
Seniority: Senior
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - NET
---

# Periferia IT — Migración VB6 a .NET 10 WinForms

## Veredicto rápido

**Postular solo si puedes demostrar experiencia real con VB6 y migración de aplicaciones de escritorio. Prioridad condicionada.**

No es una vacante web ni Full Stack: es modernización de un monolito de escritorio VB6 hacia C#/.NET 10 con WinForms. Tiene afinidad con .NET, arquitectura y bases de datos, pero se aleja del objetivo Angular + APIs + cloud.

El filtro principal parece ser experiencia de 4–5 años en VB6/.NET, stored procedures, triggers, functions y migraciones. El contrato es a plazo fijo y el sueldo no está publicado.

## Encaje estimado

| Escenario | Encaje aproximado |
|---|---:|
| VB6 profesional + migraciones + C#/WinForms | 85-95 % |
| .NET desktop sólido y capacidad de leer VB6 | 65-80 % |
| .NET web sólido, pero sin VB6/WinForms | 40-60 % |
| Sin experiencia en aplicaciones desktop legacy | Riesgo alto |

## Matriz de requisitos

| Capacidad | Importancia | Evidencia necesaria |
|---|---|---|
| VB6 | Crítica | lectura, debugging y reglas de negocio |
| C#/.NET 10 | Crítica | aplicación WinForms mantenible |
| WinForms | Crítica | UI, eventos, threading y despliegue |
| Migración legacy | Crítica | estrategia incremental y equivalencia funcional |
| Stored procedures/triggers/functions | Alta | análisis, pruebas y migración segura |
| Clean Architecture/DDD/SOLID | Alta | aplicación pragmática en desktop |
| Git | Alta | cambios pequeños, trazables y revisables |
| DevOps/CI/CD | Alta | build, tests, empaquetado y distribución |
| Contrato fijo | Condición | confirmar duración y renovación |

## VB6 que debes poder analizar

- Forms, módulos, clases y eventos;
- controles y dependencias de UI;
- COM/ActiveX y librerías externas;
- acceso a datos ADO;
- variables globales y estado compartido;
- manejo de errores `On Error`;
- archivos de configuración;
- reportes e impresión;
- automatización Office si existe;
- instalación y registro de componentes;
- dependencias entre reglas de negocio y formularios.

El objetivo inicial no es reescribir: es construir un inventario de comportamiento, dependencias, integraciones, datos y riesgos.

## Estrategia de migración

### 1. Descubrimiento

- mapa de módulos y dependencias;
- reglas de negocio;
- pantallas y flujos;
- queries/procedures/triggers;
- integraciones COM, archivos o servicios;
- procesos batch;
- usuarios, volumen y criticidad;
- errores conocidos y zonas sin uso.

### 2. Caracterización

- golden master o pruebas sobre comportamiento existente;
- datasets representativos;
- capturas de entradas/salidas;
- conciliación de resultados;
- métricas de rendimiento actuales.

### 3. Arquitectura objetivo

```text
WinForms
   │
Application / Use Cases
   │
Domain
   │
Infrastructure
   ├── database
   ├── files
   ├── COM/legacy adapters
   └── external integrations
```

### 4. Migración incremental

- identificar seams;
- extraer reglas puras;
- crear adapters alrededor de dependencias legacy;
- migrar por módulo/caso de uso;
- ejecutar VB6 y .NET en paralelo cuando sea viable;
- comparar resultados;
- habilitar rollback;
- retirar componentes solo con evidencia.

Evitar un big-bang rewrite sin pruebas, inventario ni estrategia de convivencia.

## C# y .NET 10

- nullable reference types;
- async/await y cancelación;
- DI y configuración;
- logging estructurado;
- separación UI/aplicación/dominio;
- records/value objects cuando sean apropiados;
- manejo moderno de errores;
- testing;
- rendimiento y memoria;
- interoperabilidad con COM si la transición lo exige;
- empaquetado y actualización.

No utilizar novedades de .NET solo por modernidad. Cada decisión debe mejorar mantenibilidad, seguridad o operación.

## WinForms

- ciclo de vida de forms;
- controles, eventos y data binding;
- layouts y escalado DPI;
- validación;
- navegación y estado;
- UI thread y `Invoke`;
- tareas largas sin congelar interfaz;
- cancelación y progreso;
- manejo global de errores;
- recursos/localización;
- accesibilidad;
- configuración;
- empaquetado, firma y actualización.

Separar lógica de negocio de code-behind para que pueda probarse sin abrir formularios.

## Clean Architecture, DDD y SOLID

Aplicación pragmática:

- dominio sin referencias a WinForms ni acceso a datos;
- casos de uso que coordinan operaciones;
- interfaces para persistencia/integraciones;
- adapters para ADO/COM/archivos;
- entidades y value objects para reglas importantes;
- módulos o bounded contexts si el dominio lo justifica;
- DIP para sustituir dependencias legacy;
- SRP para separar UI, reglas y datos.

No crear capas, repositorios y agregados para cada tabla sin necesidad.

Material: [[10 - Arquitectura y sistemas distribuidos]].

## Base de datos y lógica almacenada

- inventariar procedures, functions y triggers;
- identificar consumidores y dependencias;
- documentar contratos, efectos laterales y transacciones;
- analizar planes e índices;
- detectar lógica duplicada entre VB6 y base;
- crear pruebas de caracterización;
- decidir qué permanece en base y qué migra a dominio;
- versionar scripts;
- diseñar cambios backward-compatible;
- plan de rollback.

Los triggers merecen especial cuidado porque su comportamiento puede ser invisible para la aplicación nueva.

## Testing de una migración

- unit tests sobre reglas migradas;
- characterization tests del legado;
- integration tests con base;
- comparación VB6 frente a .NET con mismos datos;
- snapshot/golden master cuando sea adecuado;
- pruebas de UI para flujos críticos;
- regresión;
- performance baseline;
- pruebas de instalación, actualización y rollback;
- conciliación funcional aprobada por usuarios.

## Git, CI/CD y distribución desktop

Pipeline mínimo:

```text
pull request
  -> restore/build
  -> unit/integration tests
  -> análisis estático
  -> paquete firmado/versionado
  -> ambiente de prueba
  -> smoke test
  -> aprobación
  -> distribución controlada
  -> rollback a versión previa
```

Preparar estrategia para instalador, runtime requerido, configuración, firma de código, actualización y telemetría.

## Proyecto demostrable

Crear una mini aplicación legacy simulada:

- módulo inicial con lógica mezclada;
- characterization tests;
- extracción de dominio/casos de uso;
- WinForms como adapter de entrada;
- repositorio de base de datos;
- procedure y trigger documentados;
- migración incremental;
- comparación de resultados;
- pipeline y paquete versionado;
- ADR con decisiones y riesgos.

## Preparación de siete días

| Día | Foco | Entregable |
|---|---|---|
| 1 | CV/filtros | evidencia VB6, desktop y migraciones |
| 2 | VB6 | inventario, ADO, COM y reglas |
| 3 | WinForms/.NET | UI separada y caso de uso |
| 4 | Migración | characterization tests y strangler |
| 5 | Datos | procedures, triggers y comparación |
| 6 | Arquitectura | Clean/DDD/SOLID y ADR |
| 7 | CI/CD | paquete, instalación, rollback y entrevista |

## Preguntas para reclutamiento

1. ¿VB6 es excluyente y cuántos años específicos requieren?
2. ¿Cuál es la base de datos?
3. ¿Qué porcentaje ya fue migrado?
4. ¿La estrategia será incremental o reescritura completa?
5. ¿Existen pruebas o documentación del sistema actual?
6. ¿Qué integraciones COM/ActiveX mantiene la aplicación?
7. ¿Por qué WinForms es la arquitectura objetivo?
8. ¿Cuánto dura el contrato a plazo fijo y existe renovación?
9. ¿Qué significa el esquema híbrido 2x3?
10. ¿Cuál es el rango salarial?
11. ¿Existe soporte fuera de horario durante la migración?

## Decisión final

Es una vacante especializada con buen potencial solo si existe afinidad real con VB6/WinForms. No debe alterar la ruta principal Angular + .NET + Azure: se registra como rama condicional de modernización legacy.
