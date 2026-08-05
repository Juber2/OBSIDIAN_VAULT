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

# 04 - Stored Procedures

## Que vas a aprender

En este capitulo vas a profundizar en **Stored Procedures** de T-SQL, historicamente mucho mas centrales en el ecosistema SQL Server/.NET que en Postgres/Node: parametros de entrada y salida, y su rol en aplicaciones .NET tradicionales frente al enfoque mas moderno de Entity Framework Core.

## Explicacion

Un stored procedure (`CREATE PROCEDURE ... AS BEGIN ... END`) en T-SQL soporta parametros de entrada (`@Parametro tipo`) y parametros de **salida** (`@Parametro tipo OUTPUT`), permitiendo devolver valores calculados sin necesitar un `SELECT` de retorno explicito. Historicamente, en aplicaciones .NET tradicionales (ASP.NET clasico, pre-Entity Framework), era comun encapsular **toda** la logica de acceso a datos en stored procedures, invocados desde C# con ADO.NET directo.

Con Entity Framework Core (ver capitulo de SQL Server con .NET), la tendencia moderna es usar LINQ para la mayoria de operaciones, reservando stored procedures para casos especificos: operaciones batch complejas, logica que se beneficia de vivir cerca de los datos, o integracion con sistemas legacy que ya dependen de un catalogo de procedures existente. Es una decision arquitectonica real, no una obligacion historica a seguir sin cuestionar.

## Modelo mental

Pensa un stored procedure como una receta completa guardada en la cocina misma (la base de datos), en vez de que el mesero (la aplicacion) tenga que describir cada paso de la receta cada vez que pide un plato. Esto era la norma en la epoca donde el mesero (aplicacion .NET clasica) tenia herramientas mas limitadas para construir consultas complejas; hoy, con LINQ como herramienta mas expresiva, muchas de esas recetas se pueden describir directamente desde la mesa (el codigo C#) sin necesitar guardarlas todas en la cocina.

## Ejemplo base

```sql
CREATE PROCEDURE ObtenerResumenCliente
    @ClienteId BIGINT,
    @TotalPedidos INT OUTPUT,
    @MontoTotal DECIMAL(12,2) OUTPUT
AS
BEGIN
    SELECT @TotalPedidos = COUNT(*), @MontoTotal = SUM(Total)
    FROM Pedidos WHERE ClienteId = @ClienteId;
END;
```

```csharp
// Invocacion desde .NET con ADO.NET directo (patron tradicional)
using var cmd = new SqlCommand("ObtenerResumenCliente", connection) { CommandType = CommandType.StoredProcedure };
cmd.Parameters.AddWithValue("@ClienteId", clienteId);
var totalParam = cmd.Parameters.Add("@TotalPedidos", SqlDbType.Int); totalParam.Direction = ParameterDirection.Output;
```

## Como se usa en un proyecto real

1. Se evalua con criterio real cuando un stored procedure aporta valor (operaciones batch, logica cercana a datos) versus cuando LINQ con Entity Framework Core seria mas mantenible y testeable.
2. Se usan parametros de salida (`OUTPUT`) para devolver valores calculados sin necesitar un `SELECT` adicional.
3. Se documenta claramente cuando un proyecto .NET nuevo (como StaffTrainingPlatform) decide su estrategia: LINQ por defecto, stored procedures como excepcion justificada.
4. Se conecta esto con el capitulo de Stored Procedures de Postgres, entendiendo que ambos motores soportan el concepto, pero con un peso historico distinto en cada ecosistema.
5. Se evita mover logica de negocio compleja a stored procedures sin una razon real, aplicando el mismo criterio de "logica de negocio en la aplicacion, no escondida en la base de datos" ya visto en Postgres.

## Criterio junior

Un junior debe poder crear un stored procedure basico con parametros de entrada y salida.

## Criterio mid

Un mid debe decidir cuando un stored procedure aporta valor real frente a usar LINQ con Entity Framework Core.

## Criterio senior

Un senior debe definir la estrategia de acceso a datos del equipo (LINQ por defecto, stored procedures como excepcion documentada) para proyectos .NET nuevos.

## Errores comunes

- Usar stored procedures para toda la logica de acceso a datos "porque siempre se hizo asi", sin evaluar si LINQ con EF Core seria mas mantenible para un proyecto nuevo.
- No aprovechar parametros de salida cuando serian mas simples que un `SELECT` de retorno adicional.
- Esconder logica de negocio compleja en stored procedures, perdiendo testabilidad desde el codigo C#.
- No documentar la estrategia de acceso a datos elegida para el equipo, generando inconsistencia entre distintas partes del proyecto.

## Practica

- [ ] Crea un stored procedure con parametros de entrada y salida.
- [ ] Invoca ese procedure desde un cliente T-SQL o desde .NET (conceptualmente si no tenes el entorno completo).
- [ ] Compara la misma operacion implementada con LINQ en Entity Framework Core.
- [ ] Documenta el criterio que usarias para StaffTrainingPlatform: cuando stored procedure, cuando LINQ.
- [ ] Compara este capitulo con el ya estudiado de Stored Procedures de Postgres.

## Preguntas de repaso

- Que ventaja historica tenian los stored procedures en aplicaciones .NET tradicionales antes de Entity Framework Core?
- Como se usan parametros de salida (`OUTPUT`) en un stored procedure de T-SQL?
- Que criterio usarias para decidir entre stored procedure y LINQ para una operacion especifica?
- Por que mover logica de negocio compleja a un stored procedure puede reducir testabilidad desde C#?
