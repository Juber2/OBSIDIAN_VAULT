---
tags:
  - PostgreSQL
  - Tipos-de-Datos
  - Postgres-Avanzado
Creado: 2026-07-29
Relacionado:
  - Tipos de Datos Avanzados
  - POSTGRESQL
---

# 05 - Enum y Tipos Custom

## Que vas a aprender

En este capitulo vas a cerrar el capitulo aprendiendo **tipos ENUM** nativos de Postgres, tipos **compuestos** custom (`CREATE TYPE`), y **dominios** (`CREATE DOMAIN`), herramientas para expresar restricciones de tipo mas precisas que los tipos basicos.

## Explicacion

Un tipo **ENUM** (`CREATE TYPE nombre AS ENUM (...)`) restringe una columna a un conjunto fijo de valores validos, mas eficiente en espacio que un `text` con `CHECK IN (...)` (ya visto en el capitulo de Constraints) y auto-documentado en el esquema. El tradeoff es que **agregar un valor nuevo** a un ENUM existente requiere `ALTER TYPE ... ADD VALUE`, una operacion que en versiones antiguas de Postgres no podia revertirse dentro de la misma transaccion, aunque esto mejoro en versiones recientes.

Un **tipo compuesto** (`CREATE TYPE nombre AS (...)`) agrupa varios campos relacionados bajo un unico tipo reutilizable (como una direccion con calle, ciudad, codigo postal), util cuando ese grupo de campos se repite en multiples tablas. Un **dominio** (`CREATE DOMAIN nombre AS tipo_base CHECK (...)`) crea una variante restringida de un tipo existente (por ejemplo, un dominio `email` basado en `text` con un `CHECK` de formato), reutilizable en multiples columnas sin repetir el mismo constraint una y otra vez.

## Modelo mental

Pensa un ENUM como una lista desplegable fija en un formulario: solo podes elegir de las opciones predefinidas, nada mas. Un dominio es como una plantilla de campo reutilizable con validacion incorporada (como un campo "email" que cualquier formulario nuevo puede usar sin tener que redefinir la regla de formato cada vez desde cero).

## Ejemplo base

```sql
CREATE TYPE estado_pedido AS ENUM ('pendiente', 'pagado', 'enviado', 'cancelado');

CREATE TABLE pedidos (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    estado estado_pedido NOT NULL DEFAULT 'pendiente'
);

ALTER TYPE estado_pedido ADD VALUE 'reembolsado';   -- agregar un valor nuevo

-- Dominio reutilizable
CREATE DOMAIN email AS text CHECK (VALUE ~ '^[^@]+@[^@]+\.[^@]+$');

CREATE TABLE usuarios (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    correo email NOT NULL   -- reusa la validacion del dominio automaticamente
);

-- Tipo compuesto
CREATE TYPE direccion AS (calle text, ciudad text, codigo_postal text);

CREATE TABLE clientes (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    domicilio direccion
);
```

## Como se usa en un proyecto real

1. Se usa ENUM para columnas de estado con un conjunto de valores conocido y relativamente estable, obteniendo auto-documentacion en el esquema ademas de eficiencia de espacio.
2. Se evalua si el conjunto de valores de un ENUM realmente es estable a largo plazo; si cambia frecuentemente, un `CHECK IN (...)` con `text` puede ser mas flexible de mantener.
3. Se usa un dominio para reutilizar la misma validacion de formato (email, telefono, codigo postal) en multiples columnas sin repetir el `CHECK` cada vez.
4. Se documenta cualquier tipo custom (ENUM, compuesto, dominio) creado, dado que agrega vocabulario nuevo al esquema que un desarrollador nuevo debe entender.
5. Se conecta esto con el mapeo de tipos ENUM de TypeORM en el libro de NestJS, donde un ENUM de Postgres corresponde directamente a un ENUM de TypeScript decorado en la entidad.

## Criterio junior

Un junior debe poder crear un ENUM basico y usarlo como tipo de columna.

## Criterio mid

Un mid debe crear un dominio reutilizable para una validacion de formato repetida, y agregar valores nuevos a un ENUM existente correctamente.

## Criterio senior

Un senior debe decidir cuando un ENUM es apropiado versus un `CHECK IN (...)` con `text` (segun la estabilidad esperada del conjunto de valores), y estandarizar el uso de dominios para validaciones repetidas en todo el esquema del equipo.

## Errores comunes

- Usar ENUM para un conjunto de valores que cambia frecuentemente, sufriendo la friccion de `ALTER TYPE` en cada cambio en vez de usar `text` con `CHECK` mas flexible.
- No documentar tipos custom creados, generando vocabulario de esquema que un nuevo desarrollador no entiende sin explicacion.
- Repetir el mismo `CHECK` de formato en multiples columnas en vez de crear un dominio reutilizable.
- No conectar el ENUM de Postgres con el ENUM de TypeScript equivalente en TypeORM, generando desincronizacion entre ambos.

## Practica

- [ ] Crea un ENUM para un conjunto de estados de tu dominio y usalo en una tabla.
- [ ] Agrega un valor nuevo a ese ENUM con `ALTER TYPE`.
- [ ] Crea un dominio reutilizable para una validacion de formato que se repite en tu esquema.
- [ ] Crea un tipo compuesto para un grupo de campos relacionados que se repite entre tablas.
- [ ] Conecta un ENUM de Postgres con el decorador ENUM equivalente de TypeORM en un proyecto NestJS.

## Preguntas de repaso

- Que ventaja da un ENUM sobre un `CHECK IN (...)` con `text`, y que friccion introduce a cambio?
- Para que sirve un dominio y que problema de repeticion resuelve?
- Cuando NO convendria usar un ENUM para una columna de estado?
- Como se relaciona un ENUM de Postgres con un ENUM de TypeScript en TypeORM?
