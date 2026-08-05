---
tags:
  - SQLServer
  - PerformanceAvanzada
  - SQLServer-Performance
Creado: 2026-07-29
Relacionado:
  - Performance Avanzada
  - SQL SERVER
---

# 03 - In-Memory OLTP

## Que vas a aprender

En este capitulo vas a aprender **In-Memory OLTP** (nombre en codigo "Hekaton"), una funcionalidad sin equivalente directo en Postgres estandar: tablas que residen completamente en memoria con estructuras optimizadas para concurrencia extrema, disponible desde SQL Server 2014 en adelante.

## Explicacion

Una tabla **memory-optimized** (`CREATE TABLE ... WITH (MEMORY_OPTIMIZED = ON)`) se almacena completamente en memoria (con persistencia opcional a disco para durabilidad), usando un motor de concurrencia basado en **multi-version concurrency control optimista sin locks tradicionales** (conceptualmente relacionado con el MVCC de Postgres ya estudiado, pero implementado de forma mas agresiva, eliminando practicamente todo el overhead de locking/latching del motor de disco tradicional de SQL Server). Esto la hace apropiada para escenarios de **altisima concurrencia** con contencion severa (miles de transacciones por segundo sobre las mismas filas), donde el modelo de locking pesimista tradicional de SQL Server (ver capitulo de Transacciones) se vuelve el cuello de botella principal.

Los **Natively Compiled Stored Procedures** llevan esto un paso mas alla, compilando el procedimiento a codigo nativo C++ en vez de interpretarlo, para el maximo rendimiento posible en el path critico. In-Memory OLTP no es una solucion de uso general: se reserva para tablas especificas identificadas como cuello de botella real de concurrencia (por ejemplo, una tabla de sesiones de alta frecuencia o un contador de eventos en tiempo real), no para migrar toda la base de datos.

## Modelo mental

Pensa una tabla memory-optimized como un pizarron de mano rapida donde varias personas escriben notas simultaneamente sin tener que esperar turno formal (sin locks tradicionales), en vez del archivo fisico tradicional donde cada persona debe esperar su turno para abrir el cajon (locking pesimista en disco). Es rapidisimo, pero solo tiene sentido para el pizarron especifico de alta demanda, no para reemplazar todo el sistema de archivo de la oficina.

## Ejemplo base

```sql
-- Habilitar el filegroup memory-optimized en la base de datos
ALTER DATABASE MiApp ADD FILEGROUP MiApp_MOD CONTAINS MEMORY_OPTIMIZED_DATA;
ALTER DATABASE MiApp ADD FILE (NAME='MiApp_MOD', FILENAME='C:\Data\MiApp_MOD')
    TO FILEGROUP MiApp_MOD;

-- Tabla memory-optimized
CREATE TABLE SesionesActivas (
    SesionId INT IDENTITY PRIMARY KEY NONCLUSTERED,
    UsuarioId INT NOT NULL,
    UltimaActividad DATETIME2 NOT NULL
) WITH (MEMORY_OPTIMIZED = ON, DURABILITY = SCHEMA_AND_DATA);
```

## Como se usa en un proyecto real

1. Se identifica una tabla especifica con contencion severa de concurrencia (medida con DMVs de espera, no por sospecha) como candidata a In-Memory OLTP, no se aplica de forma general.
2. Se evalua el trade-off de memoria disponible (toda la tabla debe caber en RAM) contra el beneficio de eliminar el overhead de locking tradicional.
3. Se usa `DURABILITY = SCHEMA_AND_DATA` para mantener persistencia real de los datos, evitando `SCHEMA_ONLY` salvo en casos donde la perdida de datos ante un reinicio es genuinamente aceptable (como cache temporal).
4. Se conecta esto con el modelo de locking pesimista tradicional ya estudiado en el capitulo de Transacciones, entendiendo que In-Memory OLTP existe especificamente para evitar ese overhead en casos extremos.
5. Se documenta la decision de usar In-Memory OLTP con la evidencia de contencion real que la justifico, evitando que se convierta en una optimizacion prematura sin medicion.

## Criterio junior

Un junior debe entender que In-Memory OLTP es para casos de contencion extrema de concurrencia, no una optimizacion de uso general.

## Criterio mid

Un mid debe identificar con DMVs si una tabla especifica sufre contencion que justificaria evaluar In-Memory OLTP.

## Criterio senior

Un senior debe decidir con evidencia medida cuando In-Memory OLTP resuelve un problema real, considerando el trade-off de memoria y la complejidad operativa adicional.

## Errores comunes

- Aplicar In-Memory OLTP como optimizacion general sin medir primero si existe contencion real que lo justifique.
- Subestimar el requisito de memoria: toda la tabla memory-optimized debe caber en RAM disponible.
- Usar `DURABILITY = SCHEMA_ONLY` para datos que realmente necesitan persistir, perdiendolos ante un reinicio inesperado.
- No entender que In-Memory OLTP no reemplaza el motor de disco tradicional para el resto de la base de datos, sino que coexiste con el para tablas especificas.

## Practica

- [ ] Investiga los requisitos y limitaciones de una tabla memory-optimized.
- [ ] Crea una tabla memory-optimized de prueba y compara su performance bajo concurrencia con una tabla tradicional equivalente.
- [ ] Identifica las DMVs que usarias para medir contencion real antes de considerar In-Memory OLTP.
- [ ] Investiga la diferencia entre `DURABILITY = SCHEMA_AND_DATA` y `SCHEMA_ONLY`.
- [ ] Compara este mecanismo con el MVCC de Postgres, identificando similitudes y diferencias de implementacion.

## Preguntas de repaso

- Para que tipo de escenario esta diseñado In-Memory OLTP, especificamente?
- Que diferencia hay entre `DURABILITY = SCHEMA_AND_DATA` y `SCHEMA_ONLY`?
- Que evidencia medirias antes de decidir migrar una tabla a memory-optimized?
- Como se relaciona el mecanismo de concurrencia de In-Memory OLTP con el MVCC ya estudiado en Postgres?
