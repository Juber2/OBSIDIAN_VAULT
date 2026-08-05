---
tags:
  - PostgreSQL
  - Administracion
  - Postgres-Administracion
Creado: 2026-07-29
Relacionado:
  - Administracion y Mantenimiento
  - POSTGRESQL
---

# 04 - Usuarios Roles y Permisos

## Que vas a aprender

En este capitulo vas a aprender el sistema de **roles** de Postgres (que unifica el concepto de usuarios y grupos, distinto del modelo separado ya visto en Linux), y `GRANT`/`REVOKE` para control de acceso granular, aplicando el mismo principio de menor privilegio de todo este ecosistema de libros.

## Explicacion

En Postgres, todo es un **rol**: no hay distincion tecnica entre "usuario" y "grupo" como en Linux (ver ese libro), simplemente algunos roles tienen el atributo `LOGIN` (pueden conectarse directamente) y otros no (se usan como grupos, agrupando permisos que se otorgan a roles con login). Un rol puede pertenecer a otro rol (`GRANT rol_grupo TO rol_usuario`), heredando sus permisos.

`GRANT`/`REVOKE` otorgan y quitan permisos especificos: sobre bases de datos (`CONNECT`), esquemas (`USAGE`, `CREATE`), tablas (`SELECT`, `INSERT`, `UPDATE`, `DELETE`), e incluso columnas especificas. La practica recomendada, igual que en Linux (sudo granular) y en el resto de este ecosistema de libros, es crear roles de aplicacion con **exactamente** los permisos necesarios (por ejemplo, un rol de solo lectura para un servicio de reportes) en vez de usar el rol superusuario (`postgres`) para conexiones de aplicacion cotidianas.

## Modelo mental

Pensa un rol de Postgres como una credencial de empleado que puede tener dos formas: una version "persona real" que puede fichar entrada (LOGIN) y una version "departamento" que agrupa permisos compartidos (sin LOGIN, solo para heredar). Un empleado real puede pertenecer a uno o mas departamentos, heredando automaticamente todos los accesos que cada departamento tiene autorizados.

## Ejemplo base

```sql
CREATE ROLE app_lectura NOLOGIN;
GRANT CONNECT ON DATABASE mi_app TO app_lectura;
GRANT USAGE ON SCHEMA public TO app_lectura;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_lectura;

CREATE ROLE servicio_reportes LOGIN PASSWORD 'contraseña_segura';
GRANT app_lectura TO servicio_reportes;   -- hereda los permisos de solo lectura

CREATE ROLE app_escritura NOLOGIN;
GRANT app_lectura TO app_escritura;   -- hereda lectura, y ademas agrega escritura
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_escritura;

CREATE ROLE api_backend LOGIN PASSWORD 'otra_contraseña';
GRANT app_escritura TO api_backend;

REVOKE DELETE ON pedidos FROM app_escritura;   -- restriccion mas fina en una tabla especifica
```

## Como se usa en un proyecto real

1. Se crea un rol de aplicacion dedicado para cada servicio que se conecta a la base de datos (la API backend, un servicio de reportes de solo lectura), nunca usando el rol superusuario para conexiones cotidianas.
2. Se organizan permisos en roles "grupo" (sin LOGIN) reutilizables, y roles "usuario" (con LOGIN) que heredan de ellos, evitando otorgar permisos individualmente a cada conexion especifica.
3. Se aplica el principio de menor privilegio otorgando exactamente los permisos necesarios (por ejemplo, un servicio de reportes solo necesita `SELECT`, nunca `DELETE`).
4. Se audita periodicamente que roles tienen que permisos, similar a la auditoria de sudo/grupos ya vista en el libro de Linux.
5. Se conecta esto con la configuracion de conexion de TypeORM en NestJS, donde las credenciales usadas por la aplicacion deberian corresponder a un rol con permisos minimos, no al superusuario.

## Criterio junior

Un junior debe poder crear un rol basico con `LOGIN` y otorgar permisos simples con `GRANT`.

## Criterio mid

Un mid debe diseñar una jerarquia de roles grupo/usuario, aplicando el principio de menor privilegio a cada servicio conectado.

## Criterio senior

Un senior debe auditar y mantener la politica de roles y permisos de toda la infraestructura de bases de datos, asegurando que ninguna aplicacion de produccion use credenciales de superusuario.

## Errores comunes

- Usar el rol superusuario (`postgres`) para la conexion de la aplicacion en produccion, otorgando permisos totales innecesarios.
- Otorgar permisos individualmente a cada rol de usuario en vez de organizarlos en roles grupo reutilizables.
- No auditar periodicamente que permisos tiene cada rol, dejando acumular accesos innecesarios con el tiempo.
- No aplicar el principio de menor privilegio, otorgando `INSERT`/`UPDATE`/`DELETE` a servicios que solo necesitan `SELECT`.

## Practica

- [ ] Crea un rol grupo de solo lectura y otorga permisos `SELECT` sobre todas las tablas de un esquema.
- [ ] Crea un rol de usuario con `LOGIN` que herede de ese rol grupo.
- [ ] Crea un segundo rol grupo con permisos de escritura, heredando del de lectura.
- [ ] Restringe un permiso especifico sobre una tabla individual con `REVOKE`.
- [ ] Audita los roles y permisos de un sistema real, verificando que ninguna aplicacion use el superusuario.

## Preguntas de repaso

- Que diferencia hay entre un rol con `LOGIN` y uno sin `LOGIN` en Postgres?
- Por que organizar permisos en roles grupo reutilizables es mejor que otorgarlos individualmente a cada usuario?
- Por que una aplicacion de produccion nunca deberia conectarse con el rol superusuario?
- Como auditarias periodicamente los permisos otorgados en un sistema de bases de datos?
