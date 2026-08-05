---
tags:
  - PostgreSQL
  - Modelado
  - Postgres-Modelado
Creado: 2026-07-29
Relacionado:
  - Modelado de Datos y Diseño de Esquemas
  - POSTGRESQL
---

# 03 - Constraints CHECK UNIQUE NOT NULL

## Que vas a aprender

En este capitulo vas a aprender a usar **constraints** para forzar reglas de integridad de datos a nivel de base de datos: `NOT NULL`, `UNIQUE`, `CHECK`, y constraints multi-columna, la ultima linea de defensa que ninguna validacion de aplicacion puede saltarse.

## Explicacion

`NOT NULL` exige que una columna siempre tenga un valor. `UNIQUE` garantiza que no haya valores duplicados en una columna (o combinacion de columnas, con `UNIQUE (col1, col2)`). `CHECK (condicion)` permite definir una regla de negocio arbitraria que Postgres verifica en cada insercion o actualizacion (por ejemplo, `CHECK (precio > 0)`), rechazando cualquier operacion que la viole, sin importar que camino de codigo la genero.

La razon por la que estos constraints son valiosos, incluso cuando ya existe validacion en la capa de aplicacion (como `class-validator` en NestJS), es que son la **ultima linea de defensa**: protegen contra bugs de aplicacion, scripts de migracion mal escritos, o accesos directos a la base de datos que se saltan la logica de la API. Un constraint violado nunca deja pasar un dato invalido, sin importar por donde intento entrar.

## Modelo mental

Pensa los constraints de base de datos como el control de calidad final en una fabrica, despues de todos los controles intermedios: aunque cada estacion anterior (validacion de la API) ya revise el producto, el control final no deja pasar nada defectuoso bajo ninguna circunstancia, ni siquiera si alguien intento saltarse los controles anteriores metiendo un producto directamente a la linea de empaquetado final.

## Ejemplo base

```sql
CREATE TABLE productos (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sku text NOT NULL UNIQUE,
    nombre text NOT NULL,
    precio numeric NOT NULL CHECK (precio > 0),
    stock integer NOT NULL DEFAULT 0 CHECK (stock >= 0)
);

CREATE TABLE reservas (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sala_id bigint NOT NULL,
    fecha date NOT NULL,
    UNIQUE (sala_id, fecha)   -- constraint multi-columna: una sala no puede reservarse dos veces el mismo dia
);

ALTER TABLE productos ADD CONSTRAINT precio_razonable CHECK (precio < 1000000);
```

## Como se usa en un proyecto real

1. Se replica en la base de datos toda regla de negocio critica que ya existe en la validacion de la API (como `class-validator` en NestJS), como defensa en profundidad, aplicando el mismo principio ya visto en el capitulo de Seguridad Senior del libro de NestJS.
2. Se usan constraints `UNIQUE` multi-columna para reglas de exclusividad que involucran mas de un campo (como "una sala no puede reservarse dos veces el mismo dia").
3. Se agregan constraints `CHECK` para invariantes de negocio simples y estables (precios positivos, stock no negativo), evitando reglas demasiado complejas que serian mas apropiadas como triggers (ver capitulo siguiente).
4. Se prueba que un constraint efectivamente rechaza datos invalidos antes de confiar en el, verificando el mensaje de error que Postgres devuelve.
5. Se documenta cada constraint no obvio con un nombre descriptivo (`CONSTRAINT nombre_descriptivo CHECK (...)`) en vez de dejar que Postgres genere un nombre automatico dificil de interpretar en un error.

## Criterio junior

Un junior debe poder agregar `NOT NULL`, `UNIQUE` y `CHECK` basicos a una tabla.

## Criterio mid

Un mid debe usar constraints multi-columna para reglas de exclusividad complejas, y nombrar constraints de forma descriptiva.

## Criterio senior

Un senior debe definir que reglas de negocio deben replicarse como constraints de base de datos (defensa en profundidad) versus cuales quedan solo en la capa de aplicacion, documentando ese criterio para el equipo.

## Errores comunes

- Confiar exclusivamente en validacion de la aplicacion sin replicar reglas criticas como constraints de base de datos, dejando la puerta abierta a datos invalidos por otros caminos.
- No nombrar constraints explicitamente, generando mensajes de error crípticos dificiles de interpretar cuando algo falla.
- Agregar constraints `CHECK` demasiado complejos que serian mas apropiados como triggers o logica de aplicacion.
- No probar que un constraint realmente rechaza el dato invalido esperado antes de confiar en el.

## Practica

- [ ] Agrega `NOT NULL`, `UNIQUE` y `CHECK` a una tabla de practica.
- [ ] Crea un constraint `UNIQUE` multi-columna para una regla de exclusividad real.
- [ ] Provoca a proposito la violacion de cada constraint y observa el mensaje de error.
- [ ] Nombra explicitamente un constraint y compara el mensaje de error con uno sin nombre.
- [ ] Identifica una regla de negocio de tu proyecto que deberia replicarse como constraint de base de datos.

## Preguntas de repaso

- Por que vale la pena replicar reglas de negocio en constraints de base de datos si ya existen en la validacion de la aplicacion?
- Que tipo de regla es apropiada para un `CHECK` simple, y cual seria mejor como trigger?
- Como definirias un constraint `UNIQUE` que involucra mas de una columna?
- Por que nombrar explicitamente un constraint mejora la experiencia de debugging?
