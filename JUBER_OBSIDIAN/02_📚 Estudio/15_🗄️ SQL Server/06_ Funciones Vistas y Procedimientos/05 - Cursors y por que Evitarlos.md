---
tags:
  - SQLServer
  - Funciones
  - SQLServer-Funciones
Creado: 2026-07-29
Relacionado:
  - Funciones Vistas y Procedimientos
  - SQL SERVER
---

# 05 - Cursors y por que Evitarlos

## Que vas a aprender

En este capitulo vas a cerrar el capitulo aprendiendo **Cursors** de T-SQL, un mecanismo para procesar resultados **fila por fila** (sin equivalente de uso tan comun en el vocabulario de Postgres, donde loops similares existen en PL/pgSQL pero son igualmente desaconsejados), y por que casi siempre son la eleccion equivocada frente a operaciones basadas en conjuntos.

## Explicacion

Un cursor permite iterar sobre un resultado de consulta **fila por fila**, similar a un loop `FOR` tradicional de programacion procedural, pero SQL (tanto T-SQL como el SQL de Postgres) esta diseñado fundamentalmente para operar sobre **conjuntos completos de datos** de una vez (set-based operations), no fila por fila. Los cursors son dramaticamente mas lentos que la alternativa equivalente escrita como una unica operacion sobre el conjunto completo, dado que cada iteracion del cursor tiene overhead significativo comparado con que el motor optimice una operacion completa de una vez.

Casi cualquier logica que "parece necesitar" un cursor puede reescribirse como una operacion basada en conjuntos usando `UPDATE`/`JOIN`, `CASE WHEN`, o funciones de ventana (ver capitulo de T-SQL Avanzado), y esa reescritura casi siempre resulta en una mejora de performance dramatica. Los cursors se reservan para los pocos casos genuinamente irreductibles a logica de conjuntos (ciertas tareas administrativas complejas), no para logica de negocio de aplicacion.

## Modelo mental

Pensa un cursor como atender a una fila de 1000 personas repartiendo panfletos uno por uno, caminando individualmente hasta cada persona. Una operacion basada en conjuntos es como repartir los 1000 panfletos de una vez con una maquina automatica que los distribuye a todos simultaneamente: el resultado final es el mismo, pero la diferencia de tiempo y esfuerzo es enorme.

## Ejemplo base

```sql
-- Mal: cursor, procesa fila por fila, lento en tablas grandes
DECLARE @PedidoId BIGINT;
DECLARE cur CURSOR FOR SELECT Id FROM Pedidos WHERE Estado = 'Pendiente';
OPEN cur;
FETCH NEXT FROM cur INTO @PedidoId;
WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE Pedidos SET Estado = 'Procesado' WHERE Id = @PedidoId;
    FETCH NEXT FROM cur INTO @PedidoId;
END;
CLOSE cur; DEALLOCATE cur;

-- Bien: operacion basada en conjuntos, una unica sentencia para TODAS las filas
UPDATE Pedidos SET Estado = 'Procesado' WHERE Estado = 'Pendiente';
```

## Como se usa en un proyecto real

1. Se evita el uso de cursors por defecto, buscando siempre la equivalencia basada en conjuntos primero (un unico `UPDATE`/`INSERT`/`DELETE` que afecte todas las filas relevantes de una vez).
2. Se reserva el uso de cursors solo para las tareas administrativas genuinamente irreductibles a logica de conjuntos, documentando la justificacion.
3. Se revisa en code review cualquier uso de cursor en logica de negocio de aplicacion, cuestionando si realmente es necesario o si es un habito heredado de programacion procedural tradicional.
4. Se conecta esto con las mismas alternativas ya estudiadas en Postgres (evitar loops PL/pgSQL cuando una operacion de conjunto alcanza), reforzando el principio general de "pensar en SQL, no en loops" en cualquier motor relacional.
5. Se mide el impacto real de reemplazar un cursor por su equivalente basado en conjuntos, confirmando la mejora de performance con evidencia (Execution Plan, ver capitulo de Indices).

## Criterio junior

Un junior debe entender que los cursors son generalmente lentos y que existe una alternativa basada en conjuntos para casi cualquier caso.

## Criterio mid

Un mid debe reescribir logica basada en cursor como una operacion basada en conjuntos equivalente.

## Criterio senior

Un senior debe revisar en code review cualquier uso de cursor, estableciendo el criterio del equipo de cuando (si alguna vez) su uso es genuinamente justificado.

## Errores comunes

- Usar un cursor para logica de negocio de aplicacion que facilmente se reescribe como una operacion basada en conjuntos, sacrificando performance innecesariamente.
- No medir el impacto real de reemplazar un cursor, perdiendo la oportunidad de demostrar la mejora con evidencia concreta.
- Heredar el habito de programacion procedural (loops) sin adaptar el pensamiento al modelo declarativo de conjuntos que SQL favorece.
- No cuestionar en code review el uso de un cursor, dejando pasar codigo significativamente menos eficiente de lo necesario.

## Practica

- [ ] Escribe una logica simple usando un cursor.
- [ ] Reescribe esa misma logica como una operacion basada en conjuntos equivalente.
- [ ] Compara el Execution Plan y tiempo de ejecucion de ambas versiones.
- [ ] Identifica un uso de cursor en codigo real (si existe en tu experiencia) y evalua si podria reescribirse.
- [ ] Documenta el criterio de tu equipo sobre cuando (si alguna vez) usar cursors es aceptable.

## Preguntas de repaso

- Por que los cursors son generalmente mucho mas lentos que operaciones basadas en conjuntos equivalentes?
- Que alternativa basada en conjuntos usarias en vez de un cursor para actualizar multiples filas segun una condicion?
- Que casos genuinamente justificarian el uso de un cursor?
- Como medirias el impacto real de reemplazar un cursor por su equivalente de conjunto?

## Cierre del capitulo

Con este tema cierras el capitulo de programabilidad de SQL Server, habiendo aplicado el mismo conocimiento profundo ya construido en el libro de Postgres (funciones, vistas, triggers, procedimientos) a la sintaxis y particularidades especificas de T-SQL, sumando ademas el aprendizaje critico de pensar siempre en conjuntos, no en loops, sin importar el motor relacional que uses.
