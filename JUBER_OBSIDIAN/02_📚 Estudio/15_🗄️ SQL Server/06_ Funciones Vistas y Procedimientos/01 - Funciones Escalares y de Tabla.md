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

# 01 - Funciones Escalares y de Tabla

## Que vas a aprender

En este capitulo vas a aprender funciones en T-SQL: **funciones escalares** (devuelven un unico valor, equivalente a las funciones SQL/PL/pgSQL simples de Postgres) y **funciones de tabla** (devuelven un conjunto de filas, usable directamente en un `FROM`, sin equivalente tan directo en el vocabulario de Postgres pero con el mismo proposito).

## Explicacion

Una funcion **escalar** (`CREATE FUNCTION ... RETURNS tipo`) es identica en proposito a las funciones de Postgres ya estudiadas: encapsula un calculo reutilizable. Un detalle importante de performance: las funciones escalares en T-SQL historicamente tuvieron una reputacion de **degradar significativamente performance** cuando se usan dentro de un `WHERE` sobre muchas filas (el optimizer no siempre puede optimizarlas bien), aunque versiones modernas de SQL Server (2019+) mejoraron esto con "Scalar UDF Inlining" en muchos casos.

Una funcion **de tabla** (`RETURNS TABLE`) devuelve un conjunto de filas que se puede usar directamente como si fuera una tabla en un `FROM`, similar en espiritu a una vista parametrizada: a diferencia de una vista normal (ver tema siguiente), una funcion de tabla puede recibir **parametros**, dando mucha mas flexibilidad para encapsular logica de consulta reutilizable con inputs variables.

## Modelo mental

Pensa una funcion escalar como una formula de calculadora con nombre, identico al concepto ya visto en Postgres. Una funcion de tabla es como una vista con un control deslizante ajustable: en vez de mostrar siempre exactamente los mismos datos, podes pasarle un parametro (como un rango de fechas) y recibir un resultado tabular distinto segun ese input, algo que una vista simple no puede hacer.

## Ejemplo base

```sql
-- Funcion escalar
CREATE FUNCTION CalcularDescuento(@Precio DECIMAL(10,2), @Porcentaje DECIMAL(5,2))
RETURNS DECIMAL(10,2)
AS
BEGIN
    RETURN @Precio - (@Precio * @Porcentaje / 100);
END;

SELECT dbo.CalcularDescuento(100, 10);   -- 90

-- Funcion de tabla, parametrizada
CREATE FUNCTION ObtenerPedidosPorRango(@Desde DATE, @Hasta DATE)
RETURNS TABLE
AS
RETURN (
    SELECT * FROM Pedidos WHERE CreadoEn BETWEEN @Desde AND @Hasta
);

SELECT * FROM ObtenerPedidosPorRango('2026-01-01', '2026-06-30');
```

## Como se usa en un proyecto real

1. Se usan funciones escalares para calculos reutilizables simples, evitando su uso dentro de `WHERE` sobre tablas muy grandes en versiones anteriores a SQL Server 2019 por el riesgo de performance.
2. Se usan funciones de tabla como alternativa parametrizada a vistas simples, cuando se necesita encapsular una consulta reutilizable que varia segun input.
3. Se verifica la version de SQL Server en uso antes de asumir que "Scalar UDF Inlining" mejora automaticamente la performance de funciones escalares.
4. Se conecta esto con Postgres, donde funciones SQL simples y PL/pgSQL cubren un espacio similar, aunque sin la distincion tan marcada entre "funcion normal" y "funcion de tabla" con parametros.
5. Se documenta cualquier funcion escalar usada dentro de un `WHERE` sobre tablas grandes, como candidata a revisar si aparece un problema de performance.

## Criterio junior

Un junior debe poder crear una funcion escalar simple y una funcion de tabla basica.

## Criterio mid

Un mid debe usar funciones de tabla parametrizadas como alternativa flexible a vistas simples.

## Criterio senior

Un senior debe evaluar el impacto de performance de funciones escalares en consultas de alto volumen, considerando la version de SQL Server en uso.

## Errores comunes

- Usar una funcion escalar dentro de un `WHERE` sobre una tabla muy grande en una version antigua de SQL Server, sufriendo degradacion de performance significativa.
- No aprovechar funciones de tabla parametrizadas cuando una vista simple no puede cubrir la necesidad de recibir un input variable.
- Confundir el concepto con las funciones de Postgres sin reconocer las particularidades especificas de T-SQL (Scalar UDF Inlining, funciones de tabla).
- No verificar la version de SQL Server antes de asumir mejoras de performance de versiones recientes.

## Practica

- [ ] Crea una funcion escalar simple para un calculo reutilizable.
- [ ] Crea una funcion de tabla parametrizada y consultala directamente en un `FROM`.
- [ ] Investiga si tu version de SQL Server soporta Scalar UDF Inlining.
- [ ] Compara este concepto con las funciones ya estudiadas en Postgres.
- [ ] Identifica un caso real donde una funcion de tabla parametrizada seria mas util que una vista simple.

## Preguntas de repaso

- Que diferencia hay entre una funcion escalar y una funcion de tabla en T-SQL?
- Que riesgo de performance historico tienen las funciones escalares usadas en un `WHERE` sobre tablas grandes?
- Que ventaja da una funcion de tabla parametrizada sobre una vista simple?
- Como se relaciona este concepto con las funciones SQL/PL/pgSQL ya estudiadas en Postgres?
