---
tags:
  - SQLServer
  - TSQL-Avanzado
  - SQLServer-Avanzado
Creado: 2026-07-29
Relacionado:
  - T-SQL Avanzado
  - SQL SERVER
---

# 04 - MERGE Statement

## Que vas a aprender

En este capitulo vas a aprender el statement **`MERGE`** de T-SQL, que combina `INSERT`/`UPDATE`/`DELETE` en una unica sentencia atomica segun coincidencias entre una tabla origen y una destino, resolviendo el patron "upsert" (insertar o actualizar) de forma nativa.

## Explicacion

`MERGE INTO destino USING origen ON condicion` evalua, para cada fila del origen, si existe coincidencia en el destino: `WHEN MATCHED THEN UPDATE` (actualiza si ya existe), `WHEN NOT MATCHED THEN INSERT` (inserta si no existe), y opcionalmente `WHEN NOT MATCHED BY SOURCE THEN DELETE` (elimina del destino filas que ya no existen en el origen), todo en una **unica operacion atomica**. Esto resuelve directamente el patron "upsert" que en Postgres se maneja con `INSERT ... ON CONFLICT DO UPDATE` (una sintaxis distinta para un proposito similar, aunque `MERGE` de T-SQL es mas general y puede combinar los tres tipos de operacion simultaneamente).

Una advertencia importante: `MERGE` tuvo historicamente algunos bugs de concurrencia documentados en ciertas versiones de SQL Server bajo condiciones especificas de alta concurrencia, por lo que se recomienda usarlo con `HOLDLOCK` o verificar la documentacion de la version especifica en uso para casos criticos de alta concurrencia.

## Modelo mental

Pensa `MERGE` como sincronizar dos listas de compras: comparas la lista nueva (origen) con la lista que ya tenias (destino), y en una unica pasada decides que items agregar porque son nuevos, cuales actualizar porque cambiaron de cantidad, y opcionalmente cuales eliminar porque ya no estan en la lista nueva, todo en un unico proceso coordinado en vez de tres pasadas separadas.

## Ejemplo base

```sql
MERGE INTO Inventario AS destino
USING (SELECT ProductoId, Cantidad FROM ActualizacionesInventario) AS origen
ON destino.ProductoId = origen.ProductoId
WHEN MATCHED THEN
    UPDATE SET destino.Cantidad = origen.Cantidad
WHEN NOT MATCHED BY TARGET THEN
    INSERT (ProductoId, Cantidad) VALUES (origen.ProductoId, origen.Cantidad)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

```sql
-- Equivalente conceptual en Postgres (ver libro complementario)
INSERT INTO inventario (producto_id, cantidad)
VALUES (1, 100)
ON CONFLICT (producto_id) DO UPDATE SET cantidad = EXCLUDED.cantidad;
```

## Como se usa en un proyecto real

1. Se usa `MERGE` para operaciones de sincronizacion de datos (importaciones, ETL simple) donde se necesita insertar, actualizar y opcionalmente eliminar en una unica operacion atomica.
2. Se verifica la version de SQL Server y se considera `HOLDLOCK` para casos de alta concurrencia, dado el historial de bugs documentados en versiones especificas.
3. Se compara esta sintaxis con `INSERT ... ON CONFLICT` de Postgres al trabajar en ambos motores, entendiendo que resuelven necesidades similares con enfoques distintos.
4. Se prueba exhaustivamente cualquier uso de `MERGE` en produccion, dado su historial de comportamiento inesperado bajo ciertas condiciones documentadas.
5. Se documenta claramente la logica de `MERGE` compleja, dado que combina tres operaciones en una unica sentencia que puede ser dificil de leer rapidamente.

## Criterio junior

Un junior debe poder escribir un `MERGE` basico que combine `INSERT` y `UPDATE` segun coincidencia.

## Criterio mid

Un mid debe usar `MERGE` para sincronizacion de datos completa (insertar, actualizar, eliminar), y conocer la clausula `HOLDLOCK` para casos de concurrencia.

## Criterio senior

Un senior debe evaluar el historial de bugs documentados de `MERGE` en la version especifica de SQL Server en uso antes de confiar en el para operaciones criticas de alta concurrencia.

## Errores comunes

- Usar `MERGE` sin `HOLDLOCK` en escenarios de alta concurrencia donde el historial de bugs documentados podria causar comportamiento inesperado.
- No probar exhaustivamente la logica completa de `MERGE` (los tres casos: matched, not matched by target, not matched by source).
- Confundir la sintaxis de `MERGE` con `ON CONFLICT` de Postgres sin entender las diferencias reales de capacidad entre ambos.
- No documentar la logica compleja de un `MERGE` con multiples clausulas `WHEN`, dificultando su mantenimiento.

## Practica

- [ ] Escribe un `MERGE` que combine `INSERT` y `UPDATE` para un caso de sincronizacion simple.
- [ ] Agrega la clausula `WHEN NOT MATCHED BY SOURCE THEN DELETE` para completar la sincronizacion.
- [ ] Investiga la clausula `HOLDLOCK` y cuando aplicarla.
- [ ] Compara este `MERGE` con el equivalente `INSERT ... ON CONFLICT` de Postgres para el mismo caso de uso.
- [ ] Documenta un caso real de tu proyecto donde `MERGE` resolveria una necesidad de sincronizacion de datos.

## Preguntas de repaso

- Que tres operaciones puede combinar `MERGE` en una unica sentencia atomica?
- Que precaucion tomarias con `MERGE` en escenarios de alta concurrencia?
- Como se compara `MERGE` de T-SQL con `INSERT ... ON CONFLICT` de Postgres?
- Que caso de tu proyecto real se beneficiaria de usar `MERGE`?
