---
tags:
  - SQLServer
  - Performance
  - SQLServer-Indices
Creado: 2026-07-29
Relacionado:
  - Indices y Optimizacion de Queries
  - SQL SERVER
---

# 01 - Indices Clustered y Non-Clustered

## Que vas a aprender

En este capitulo vas a aprender la distincion mas importante de indices en SQL Server, sin equivalente directo tan marcado en Postgres: **indice clustered** (define el orden fisico real de los datos en disco) versus **indice non-clustered** (una estructura separada que apunta hacia los datos).

## Explicacion

Un **indice clustered** determina el **orden fisico real** en que las filas se almacenan en disco: una tabla puede tener **como maximo un unico** indice clustered (porque los datos solo pueden estar fisicamente ordenados de una unica forma a la vez), tipicamente creado automaticamente sobre la clave primaria. Esto es fundamentalmente distinto de Postgres, donde **ningun** indice determina el orden fisico de los datos (todos los indices de Postgres, incluyendo el B-tree sobre la clave primaria, son estructuras separadas que apuntan hacia la tabla heap).

Un **indice non-clustered** es una estructura separada (similar en concepto al B-tree de Postgres) que contiene una copia ordenada de las columnas indexadas mas un puntero hacia la fila real: una tabla puede tener **multiples** indices non-clustered. Elegir bien que columna(s) llevan el indice clustered (generalmente la clave primaria, pero no siempre la eleccion optima) es una decision de diseño fundamental especifica de SQL Server que no tiene equivalente directo en el modelo de Postgres.

## Modelo mental

Pensa el indice clustered como el orden fisico real en que los libros estan ubicados en los estantes de una biblioteca (solo puede haber un unico orden fisico posible a la vez). Un indice non-clustered es como un catalogo de fichas separado, ordenado por un criterio distinto (por ejemplo, por autor), donde cada ficha te dice exactamente en que estante fisico buscar el libro real. Postgres, en cambio, no tiene el concepto de "estante ordenado fisicamente"; todos sus indices son mas parecidos a catalogos de fichas separados, sin importar cual sea.

## Ejemplo base

```sql
CREATE TABLE Pedidos (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,   -- crea automaticamente un indice CLUSTERED sobre Id
    ClienteId BIGINT NOT NULL,
    Estado NVARCHAR(20)
);

-- Indice NON-CLUSTERED adicional sobre una columna consultada frecuentemente
CREATE NONCLUSTERED INDEX IX_Pedidos_ClienteId ON Pedidos (ClienteId);

-- Ver que indices tiene una tabla y su tipo
SELECT i.name, i.type_desc
FROM sys.indexes i
WHERE i.object_id = OBJECT_ID('Pedidos');
```

## Como se usa en un proyecto real

1. Se deja el indice clustered por defecto sobre la clave primaria autoincremental en la mayoria de tablas, siendo una eleccion razonable para la mayoria de casos.
2. Se evalua un indice clustered sobre una columna distinta a la clave primaria cuando el patron de acceso dominante de la tabla realmente lo justifica (por ejemplo, una columna de fecha consultada por rango frecuentemente).
3. Se agregan indices non-clustered sobre columnas de filtro y join frecuentes, aplicando el mismo criterio de indexacion ya visto en Postgres, pero con esta distincion adicional especifica de SQL Server.
4. Se conecta esto con las migrations de Entity Framework Core (ver capitulo de SQL Server con .NET), donde `HasKey()` genera el indice clustered por defecto sobre la clave primaria.
5. Se documenta la eleccion de indice clustered cuando no es la clave primaria por defecto, dado que es una decision no obvia que un nuevo desarrollador podria no anticipar.

## Criterio junior

Un junior debe entender la diferencia conceptual entre indice clustered y non-clustered, y saber que solo puede existir un clustered por tabla.

## Criterio mid

Un mid debe crear indices non-clustered apropiados para columnas de filtro frecuente, entendiendo que complementan al clustered existente.

## Criterio senior

Un senior debe decidir cuando el indice clustered por defecto (clave primaria) no es la eleccion optima para el patron de acceso real de una tabla especifica.

## Errores comunes

- Confundir el modelo de indices de SQL Server con el de Postgres, asumiendo que todos los indices son equivalentes a B-tree simples sin la distincion fisica del clustered.
- Crear multiples indices clustered en la misma tabla, lo cual no es posible y genera un error.
- No considerar si la clave primaria autoincremental por defecto realmente es la mejor eleccion de indice clustered para el patron de acceso real.
- No documentar una eleccion no obvia de indice clustered, generando confusion para futuros mantenedores.

## Practica

- [ ] Crea una tabla y verifica que su clave primaria genero automaticamente un indice clustered.
- [ ] Agrega un indice non-clustered sobre una columna de filtro frecuente.
- [ ] Consulta `sys.indexes` para ver el tipo de cada indice de una tabla.
- [ ] Compara conceptualmente este modelo con el de Postgres, donde no existe la distincion clustered/non-clustered de la misma forma.
- [ ] Evalua si alguna tabla de un proyecto real se beneficiaria de un indice clustered distinto a la clave primaria.

## Preguntas de repaso

- Que determina un indice clustered que ningun indice de Postgres determina de la misma forma?
- Por que una tabla solo puede tener un unico indice clustered?
- Cuantos indices non-clustered puede tener una tabla?
- Cuando se justificaria elegir un indice clustered distinto a la clave primaria por defecto?
