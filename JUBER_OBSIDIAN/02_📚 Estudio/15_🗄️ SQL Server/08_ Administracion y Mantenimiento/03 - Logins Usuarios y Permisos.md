---
tags:
  - SQLServer
  - Administracion
  - SQLServer-Administracion
Creado: 2026-07-29
Relacionado:
  - Administracion y Mantenimiento
  - SQL SERVER
---

# 03 - Logins Usuarios y Permisos

## Que vas a aprender

En este capitulo vas a aprender el modelo de seguridad de SQL Server: **Logins** (a nivel de servidor) versus **Users** (a nivel de base de datos), y `GRANT`/`DENY`/`REVOKE` para permisos, aplicando el mismo principio de menor privilegio ya establecido consistentemente en toda esta serie de libros.

## Explicacion

SQL Server separa la autenticacion en dos niveles, a diferencia del modelo unificado de roles de Postgres (ver ese libro): un **Login** existe a nivel de **servidor completo** (puede autenticarse en la instancia), y un **User** existe a nivel de **base de datos especifica**, mapeado a un Login para determinar que puede hacer dentro de esa base en particular. Un mismo Login puede mapearse a Users en multiples bases de datos distintas de la misma instancia.

Ademas de `GRANT` (otorgar) y `REVOKE` (quitar un otorgamiento previo, dejando neutral), SQL Server tiene **`DENY`**, sin equivalente directo en Postgres: `DENY` **prohibe explicitamente** un permiso, y esa prohibicion **tiene prioridad sobre cualquier `GRANT`** que ese usuario reciba de cualquier otro origen (incluso roles a los que pertenezca), una herramienta poderosa para bloquear un permiso especifico de forma absoluta e innegociable.

## Modelo mental

Pensa un Login como el pase de acceso general al edificio completo (el servidor), y un User como la credencial especifica que determina a que oficinas (bases de datos) y con que permisos podes entrar una vez adentro del edificio. `DENY` es como una prohibicion especial escrita directamente en tu ficha personal que ningun grupo o comite (rol) al que pertenezcas puede anular, a diferencia de `REVOKE` que simplemente borra un permiso previamente otorgado sin dejar ninguna prohibicion activa.

## Ejemplo base

```sql
-- Login a nivel de servidor
CREATE LOGIN AppLogin WITH PASSWORD = 'ContraseñaSegura123!';

-- User a nivel de base de datos, mapeado a ese Login
USE MiApp;
CREATE USER AppUser FOR LOGIN AppLogin;

-- Permisos granulares
GRANT SELECT, INSERT, UPDATE ON SCHEMA::dbo TO AppUser;
DENY DELETE ON Pedidos TO AppUser;   -- prohibicion absoluta, sin importar otros GRANT

-- Roles de base de datos, similar en espiritu a roles grupo de Postgres
ALTER ROLE db_datareader ADD MEMBER AppUser;
```

## Como se usa en un proyecto real

1. Se crea un Login/User dedicado para la conexion de la aplicacion .NET (StaffTrainingPlatform), nunca usando el usuario administrador `sa` para conexiones cotidianas, aplicando el mismo principio de menor privilegio ya visto en Postgres.
2. Se usa `DENY` para prohibiciones absolutas criticas (como impedir `DELETE` en tablas de auditoria) que no deben poder ser anuladas accidentalmente por membresia futura en otro rol.
3. Se organizan permisos con roles de base de datos (`db_datareader`, `db_datawriter`, o roles custom) en vez de otorgar permisos individualmente a cada usuario.
4. Se conecta esto con la cadena de conexion de Entity Framework Core (ver capitulo de SQL Server con .NET), asegurando que use un usuario con permisos minimos apropiados.
5. Se audita periodicamente Logins y Users existentes, el mismo proceso ya recomendado para roles de Postgres.

## Criterio junior

Un junior debe poder crear un Login y un User basico, entendiendo la diferencia entre ambos niveles.

## Criterio mid

Un mid debe usar `GRANT`/`DENY`/`REVOKE` con criterio, entendiendo que `DENY` tiene prioridad absoluta sobre cualquier `GRANT`.

## Criterio senior

Un senior debe auditar y mantener la politica de seguridad de bases de datos SQL Server de toda la organizacion, asegurando que ninguna aplicacion use credenciales administrativas.

## Errores comunes

- Usar el usuario administrador (`sa`) para la conexion de la aplicacion en produccion, otorgando permisos totales innecesarios.
- Confundir `REVOKE` (neutral, sin prohibicion activa) con `DENY` (prohibicion absoluta que prevalece sobre otros GRANT).
- No separar correctamente Login (nivel servidor) de User (nivel base de datos), generando confusion en el modelo de seguridad.
- No auditar periodicamente que Logins/Users tienen que permisos.

## Practica

- [ ] Crea un Login y un User mapeado, aplicando el modelo de dos niveles de SQL Server.
- [ ] Otorga permisos granulares con `GRANT` y prueba una prohibicion absoluta con `DENY`.
- [ ] Verifica que `DENY` prevalece incluso si el usuario pertenece a un rol con `GRANT` para el mismo permiso.
- [ ] Organiza permisos usando roles de base de datos en vez de otorgamiento individual.
- [ ] Compara este modelo de dos niveles con el modelo unificado de roles de Postgres.

## Preguntas de repaso

- Que diferencia hay entre un Login y un User en el modelo de seguridad de SQL Server?
- Que prioridad tiene `DENY` frente a un `GRANT` del mismo permiso otorgado por otro origen?
- Por que una aplicacion de produccion nunca deberia conectarse con el usuario `sa`?
- Como se compara este modelo de dos niveles con el modelo unificado de roles de Postgres?
