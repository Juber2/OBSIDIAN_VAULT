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

# 02 - Claves Primarias y Foraneas

## Que vas a aprender

En este capitulo vas a profundizar en **claves primarias** (incluyendo la eleccion entre `bigint` autoincremental vs UUID) y **claves foraneas** con sus politicas de `ON DELETE`/`ON UPDATE`, la base de la integridad referencial.

## Explicacion

Una clave primaria identifica de forma unica cada fila de una tabla. La eleccion entre **`bigint` autoincremental** (secuencial, mas eficiente para indices y joins, pero revela informacion sobre el volumen de datos y el orden de creacion) y **UUID** (aleatorio, no revela informacion, generable del lado del cliente antes de insertar, pero mas pesado para indices por su tamaño y falta de orden secuencial) es una decision de diseño real con tradeoffs, no una eleccion arbitraria.

Una **clave foranea** (`REFERENCES`) garantiza integridad referencial: no se puede insertar un valor que no exista en la tabla referenciada, y las politicas `ON DELETE`/`ON UPDATE` definen que pasa cuando la fila referenciada se elimina o modifica: `CASCADE` (propaga el cambio), `RESTRICT`/`NO ACTION` (impide la operacion si hay referencias), `SET NULL` (deja la referencia en null). Elegir la politica correcta segun la semantica real del negocio es critico para evitar tanto perdida accidental de datos como bloqueos inesperados.

## Modelo mental

Pensa una clave foranea como una regla de la biblioteca que dice "no podes anotar en el registro de prestamos un libro que no existe en el catalogo". Las politicas `ON DELETE` son la regla de que hacer si ese libro se da de baja del catalogo mientras todavia hay prestamos activos referenciandolo: tirar tambien esos prestamos (`CASCADE`), prohibir dar de baja el libro mientras existan prestamos (`RESTRICT`), o dejar el prestamo pero sin saber a que libro se referia (`SET NULL`).

## Ejemplo base

```sql
CREATE TABLE clientes (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre text NOT NULL
);

CREATE TABLE pedidos (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    cliente_id bigint NOT NULL REFERENCES clientes(id) ON DELETE RESTRICT,
    total numeric NOT NULL
);

CREATE TABLE pedido_items (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    pedido_id bigint NOT NULL REFERENCES pedidos(id) ON DELETE CASCADE,
    -- items de un pedido no tienen sentido sin el pedido: se eliminan junto con el
    producto text,
    cantidad integer
);

-- UUID como alternativa de clave primaria
CREATE TABLE sesiones (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id bigint REFERENCES clientes(id)
);
```

## Como se usa en un proyecto real

1. Se usa `bigint` autoincremental como default para la mayoria de tablas, reservando UUID para casos donde se necesita generar el ID del lado del cliente antes de insertar, o donde no revelar volumen/orden es un requisito real de seguridad.
2. Se elige `ON DELETE CASCADE` solo cuando la relacion es de "composicion real" (los items de un pedido no existen sin el pedido); se usa `RESTRICT` cuando la eliminacion accidental de la entidad padre no deberia arrastrar datos relacionados sin una decision explicita.
3. Se define siempre una politica `ON DELETE` explicita en vez de dejar el comportamiento por defecto (`NO ACTION`), documentando la intencion real de negocio.
4. Se conecta esto con las relaciones de TypeORM del libro de NestJS, donde `cascade` y `onDelete` mapean directamente a estos mismos conceptos de SQL.
5. Se revisa cuidadosamente el orden de eliminacion en pruebas antes de aplicar `CASCADE` en produccion, dado que puede eliminar mas datos de los esperados si la cadena de referencias es mas profunda de lo anticipado.

## Criterio junior

Un junior debe poder definir una clave primaria y una clave foranea basica con `REFERENCES`.

## Criterio mid

Un mid debe elegir la politica `ON DELETE` correcta segun la semantica real de cada relacion, y justificar la eleccion entre `bigint` y UUID para una clave primaria.

## Criterio senior

Un senior debe diseñar la estrategia de claves primarias de todo el sistema (bigint vs UUID segun contexto) y definir convenciones de politicas `ON DELETE` para el equipo, revisando en code review cualquier `CASCADE` no justificado.

## Errores comunes

- Usar `ON DELETE CASCADE` sin pensar en la cadena completa de eliminaciones que podria disparar, arriesgando perdida de datos mas amplia de lo esperado.
- Dejar el comportamiento por defecto (`NO ACTION`) sin decidir conscientemente la politica correcta, generando errores de foreign key confusos al intentar eliminar.
- Elegir UUID por defecto sin considerar el costo de indices mas pesados y peor localidad de datos que implica frente a un ID secuencial.
- No validar en pruebas el efecto real de un `CASCADE` antes de aplicarlo en produccion.

## Practica

- [ ] Define una relacion con clave foranea usando `ON DELETE CASCADE` y otra con `ON DELETE RESTRICT`, probando el efecto de cada una.
- [ ] Compara la definicion de una clave primaria `bigint` autoincremental con una `uuid`.
- [ ] Documenta el criterio que usarias para elegir entre ambas en un proyecto real.
- [ ] Conecta la politica `ON DELETE` de una relacion con la opcion `cascade`/`onDelete` equivalente en TypeORM.
- [ ] Revisa un esquema real y verifica que cada clave foranea tiene una politica `ON DELETE` explicita y justificada.

## Preguntas de repaso

- Que tradeoff real existe entre usar `bigint` autoincremental y UUID como clave primaria?
- Que diferencia de comportamiento hay entre `ON DELETE CASCADE` y `ON DELETE RESTRICT`?
- Por que es importante definir explicitamente una politica `ON DELETE` en vez de dejar el comportamiento por defecto?
- Que riesgo tiene aplicar `CASCADE` sin haber probado el efecto completo de la cadena de eliminaciones?
