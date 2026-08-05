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

# 01 - Funciones SQL y PLpgSQL

## Que vas a aprender

En este capitulo vas a aprender a escribir **funciones** en Postgres: funciones SQL simples (una sola consulta), y funciones **PL/pgSQL** (el lenguaje procedural nativo de Postgres, con variables, condicionales y loops, similar en espiritu a lo ya visto en Bash Scripting del libro de Linux).

## Explicacion

Una funcion SQL simple (`LANGUAGE sql`) envuelve una unica consulta reutilizable, util para encapsular logica de calculo repetida. Una funcion **PL/pgSQL** (`LANGUAGE plpgsql`) permite logica procedural completa: declarar variables, usar `IF`/`ELSE`, loops (`FOR`, `WHILE`), manejo de excepciones, y multiples sentencias SQL en secuencia, siendo el lenguaje mas usado para logica de negocio compleja que vive directamente en la base de datos.

La decision de que logica vive en la base de datos (funciones/triggers) versus en la aplicacion (NestJS) es una decision arquitectonica real: la logica en la base de datos es mas rapida (sin latencia de red) y garantiza consistencia sin importar el camino de acceso, pero es mas dificil de testear, versionar, y menos portable entre motores, mientras que la logica en la aplicacion es mas facil de mantener con las herramientas del ecosistema Node/TypeScript ya conocidas.

## Modelo mental

Pensa una funcion SQL simple como una formula de calculadora guardada con nombre, que siempre da el mismo tipo de resultado para los mismos inputs. Una funcion PL/pgSQL es como un pequeño programa completo que vive dentro de la base de datos misma, capaz de tomar decisiones (`IF`), repetir pasos (loops), y manejar situaciones inesperadas (excepciones), sin necesitar salir nunca del motor de base de datos para hacerlo.

## Ejemplo base

```sql
-- Funcion SQL simple
CREATE FUNCTION calcular_descuento(precio numeric, porcentaje numeric)
RETURNS numeric AS $$
    SELECT precio - (precio * porcentaje / 100)
$$ LANGUAGE sql IMMUTABLE;

SELECT calcular_descuento(100, 10);   -- 90

-- Funcion PL/pgSQL con logica procedural
CREATE FUNCTION obtener_categoria_cliente(total_gastado numeric)
RETURNS text AS $$
BEGIN
    IF total_gastado > 10000 THEN
        RETURN 'VIP';
    ELSIF total_gastado > 1000 THEN
        RETURN 'Frecuente';
    ELSE
        RETURN 'Regular';
    END IF;
END;
$$ LANGUAGE plpgsql;
```

## Como se usa en un proyecto real

1. Se usan funciones SQL simples para calculos reutilizables que se repiten en multiples consultas, evitando duplicar la misma expresion en varios lugares.
2. Se usan funciones PL/pgSQL para logica que realmente necesita procedimiento (condicionales, loops) y que se beneficia de vivir cerca de los datos, como validaciones complejas usadas por multiples triggers.
3. Se marca una funcion como `IMMUTABLE` cuando siempre devuelve el mismo resultado para los mismos argumentos (permitiendo que Postgres la optimice, incluso indexandola), `STABLE` cuando no cambia dentro de una misma consulta pero puede variar entre consultas, y `VOLATILE` (default) cuando puede cambiar en cualquier momento.
4. Se decide conscientemente que logica de negocio vive en la base de datos versus en NestJS, documentando esa decision como parte de la arquitectura del sistema.
5. Se testean funciones PL/pgSQL con casos de prueba SQL directos, dado que no se benefician automaticamente del framework de testing de la aplicacion (ver capitulo de Testing del libro de NestJS).

## Criterio junior

Un junior debe poder escribir una funcion SQL simple reutilizable para un calculo repetido.

## Criterio mid

Un mid debe escribir funciones PL/pgSQL con condicionales y loops, y marcar correctamente la volatilidad (`IMMUTABLE`/`STABLE`/`VOLATILE`) de cada funcion.

## Criterio senior

Un senior debe decidir con criterio arquitectonico que logica de negocio vive en funciones de base de datos versus en la capa de aplicacion, documentando esa decision y sus tradeoffs.

## Errores comunes

- No marcar correctamente la volatilidad de una funcion, perdiendo oportunidades de optimizacion del planner o, peor, permitiendo optimizaciones incorrectas sobre una funcion mal marcada.
- Mover demasiada logica de negocio a la base de datos sin considerar el costo de testabilidad y portabilidad que eso implica.
- Duplicar la misma expresion de calculo en multiples consultas en vez de encapsularla en una funcion SQL reutilizable.
- No testear funciones PL/pgSQL con casos de prueba explicitos, confiando en que "funcionan" sin verificacion sistematica.

## Practica

- [ ] Escribe una funcion SQL simple para un calculo que se repite en tu proyecto.
- [ ] Escribe una funcion PL/pgSQL con al menos un condicional y un loop.
- [ ] Marca correctamente la volatilidad de cada funcion que escribas.
- [ ] Escribe casos de prueba SQL directos para verificar el comportamiento de una funcion PL/pgSQL.
- [ ] Documenta un criterio propio de cuando logica de negocio deberia vivir en la base de datos versus en NestJS.

## Preguntas de repaso

- Que diferencia hay entre una funcion SQL simple y una funcion PL/pgSQL?
- Que significa marcar una funcion como `IMMUTABLE`, y que beneficio de optimizacion da?
- Que tradeoff real existe entre poner logica de negocio en la base de datos versus en la aplicacion?
- Como testearias una funcion PL/pgSQL sin depender del framework de testing de la aplicacion?
