---
tags:
  - SQLServer
  - SeniorSQLServer
  - SQLServer-Senior
Creado: 2026-07-29
Relacionado:
  - Senior SQL Server
  - SQL SERVER
---

# 04 - Code Review de T-SQL

## Que vas a aprender

En este capitulo vas a aprender un checklist sistematico para revisar codigo T-SQL en Pull Requests (stored procedures, migraciones, queries LINQ complejas), consolidando en una lista practica todo lo estudiado a lo largo de este libro.

## Explicacion

Un code review efectivo de T-SQL no busca solo errores de sintaxis (que el compilador ya detecta), sino problemas de diseño que solo se manifiestan bajo carga real de produccion: uso de cursores donde una operacion basada en conjuntos seria mas eficiente (ver capitulo de T-SQL Avanzado), Dynamic SQL construido con concatenacion de strings en vez de `sp_executesql` parametrizado (riesgo de SQL injection), falta de manejo de transacciones apropiado en operaciones multi-paso, indices faltantes o redundantes evidenciados por el Execution Plan, y cambios de esquema potencialmente bloqueantes sin el patron multi-paso necesario (ver tema anterior).

Un revisor senior tambien evalua el **nivel de aislamiento** implicito de las queries dentro de una transaccion (ver capitulo de Transacciones), si `NOLOCK`/`READ UNCOMMITTED` se esta usando apropiadamente o como un parche riesgoso para "arreglar" lentitud sin entender la causa raiz (introduciendo dirty reads), y si las pruebas asociadas al cambio (ver capitulo de Testing con SQL Server) usan una base de datos real via Testcontainers en vez de mocks que ocultarian problemas de comportamiento especifico del motor.

## Modelo mental

Pensa el code review de T-SQL como una auditoria de seguridad e ingenieria combinada: no solo revisas que la puerta cierre (que el codigo compile y funcione en el caso feliz), sino que resista un terremoto (carga real de produccion), que nadie pueda forzarla desde afuera (SQL injection), y que la remodelacion no derrumbe el edificio mientras hay gente adentro (migraciones bloqueantes).

## Ejemplo base

```sql
-- Señal de alerta en code review: cursor donde un JOIN basado en conjuntos seria mejor
-- (ver tema de T-SQL Avanzado para la alternativa correcta)
DECLARE cursor_cliente CURSOR FOR SELECT Id FROM Clientes;
-- ...

-- Señal de alerta: Dynamic SQL con concatenacion, vulnerable a SQL injection
SET @sql = 'SELECT * FROM Cursos WHERE Nombre = ''' + @nombreUsuario + '''';

-- Version correcta que un reviewer deberia exigir
EXEC sp_executesql N'SELECT * FROM Cursos WHERE Nombre = @Nombre',
    N'@Nombre NVARCHAR(100)', @Nombre = @nombreUsuario;

-- Señal de alerta: NOLOCK usado como parche sin entender el riesgo de dirty reads
SELECT * FROM Pedidos WITH (NOLOCK) WHERE ClienteId = 5;
```

## Como se usa en un proyecto real

1. Se aplica este checklist en cada Pull Request de StaffTrainingPlatform que modifica esquema, stored procedures, o queries LINQ complejas que generan T-SQL no trivial.
2. Se exige reemplazar cursores por operaciones basadas en conjuntos salvo justificacion explicita documentada de por que un cursor es genuinamente necesario en ese caso especifico.
3. Se rechaza cualquier Dynamic SQL construido con concatenacion de strings de entrada de usuario, exigiendo `sp_executesql` parametrizado o `FromSqlInterpolated` desde EF Core.
4. Se verifica que cambios de esquema sobre tablas grandes sigan el patron multi-paso o usen `WITH (ONLINE = ON)` cuando corresponda, evitando bloqueos inesperados en produccion.
5. Se confirma que las pruebas asociadas usan Testcontainers con SQL Server real (ver capitulo de Testing), no el In-Memory Provider, para cambios que afectan comportamiento real de la capa de datos.

## Criterio junior

Un junior debe poder identificar Dynamic SQL con concatenacion insegura y cursores evidentemente innecesarios en una revision de codigo.

## Criterio mid

Un mid debe evaluar el Execution Plan de una query modificada y detectar indices faltantes o redundantes durante el review.

## Criterio senior

Un senior debe conducir reviews que evaluen niveles de aislamiento, riesgo de locking en migraciones, y la estrategia de testing asociada a cada cambio de T-SQL.

## Errores comunes

- Aprobar Dynamic SQL con concatenacion de strings sin exigir `sp_executesql` parametrizado, dejando pasar una vulnerabilidad de SQL injection.
- No revisar el Execution Plan de queries modificadas, dejando pasar regresiones de performance que solo se manifiestan con datos reales de produccion.
- Aceptar `NOLOCK` como solucion a lentitud sin entender ni documentar el riesgo real de dirty reads que introduce.
- No exigir el patron multi-paso para cambios de esquema riesgosos, permitiendo migraciones bloqueantes en produccion.

## Practica

- [ ] Diseña un checklist propio de code review de T-SQL basado en los temas cubiertos en este libro.
- [ ] Revisa un stored procedure de ejemplo (o real) buscando cursores reemplazables por operaciones basadas en conjuntos.
- [ ] Identifica Dynamic SQL inseguro en un fragmento de codigo de ejemplo y corrigelo con `sp_executesql`.
- [ ] Evalua si una migracion de ejemplo sigue el patron multi-paso apropiado para su nivel de riesgo.
- [ ] Aplica este checklist a un Pull Request real (o hipotetico) de StaffTrainingPlatform.

## Preguntas de repaso

- Que riesgo especifico introduce Dynamic SQL construido con concatenacion de strings?
- Por que `NOLOCK` deberia tratarse como una decision consciente y documentada, no un parche automatico?
- Que verificarias en el Execution Plan de una query modificada durante un code review?
- Por que las pruebas asociadas a un cambio de T-SQL deberian usar SQL Server real via Testcontainers en vez de mocks?
