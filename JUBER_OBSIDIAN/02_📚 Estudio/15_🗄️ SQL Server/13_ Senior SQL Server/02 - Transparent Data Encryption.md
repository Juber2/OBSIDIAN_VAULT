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

# 02 - Transparent Data Encryption

## Que vas a aprender

En este capitulo vas a aprender **Transparent Data Encryption (TDE)**, cifrado de los archivos fisicos de la base de datos (`.mdf`/`.ldf`) en reposo, completamente transparente para la aplicacion, sin equivalente nativo directo en Postgres estandar (que requiere cifrado a nivel de sistema de archivos o disco externo al motor).

## Explicacion

TDE cifra los archivos de datos y log **en el nivel de pagina fisica**, de forma totalmente transparente para las queries y la aplicacion (ningun cambio de codigo necesario), protegiendo especificamente contra el escenario de que alguien robe fisicamente el archivo `.mdf` (o un backup `.bak`) e intente restaurarlo en otro servidor sin autorizacion: sin la clave de cifrado (protegida por un certificado en el servidor original), el archivo robado es inutilizable. Es importante entender que TDE **no protege datos en transito por la red** (para eso se usa `Encrypt=True` en la cadena de conexion, ver capitulo de .NET) ni protege contra un usuario autorizado que consulta la base de datos activa y en ejecucion (para eso existen RLS, Column-Level Encryption, o Dynamic Data Masking).

La jerarquia de claves de TDE es: **Service Master Key** (a nivel de instancia) protege la **Database Master Key** (a nivel de base de datos `master`), que protege un **Certificado**, que protege la **Database Encryption Key (DEK)** de la base de datos especifica que se cifra. Es critico respaldar el certificado y su clave privada de forma segura y separada de los backups de la base de datos: sin ese certificado, un backup cifrado con TDE es **irrecuperable**, incluso para el propio equipo que lo genero.

## Modelo mental

Pensa TDE como guardar los archivos fisicos de la base de datos dentro de una caja fuerte cerrada con llave: cualquiera puede robar la caja fuerte completa (el archivo `.mdf` o el backup), pero sin la llave correcta (el certificado), el contenido sigue siendo inutilizable. Es fundamental no perder esa llave por separado: si perdes la llave, ni siquiera vos mismo podes volver a abrir tu propia caja fuerte.

## Ejemplo base

```sql
-- Crear la Database Master Key (a nivel de base de datos master)
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'ContraseñaMuySegura123!';

-- Crear el certificado que protegera la DEK
CREATE CERTIFICATE CertTDE_MiApp WITH SUBJECT = 'Certificado TDE para MiApp';

-- Respaldar el certificado (CRITICO: guardar en ubicacion separada y segura)
BACKUP CERTIFICATE CertTDE_MiApp
    TO FILE = 'C:\CertBackup\CertTDE_MiApp.cer'
    WITH PRIVATE KEY (
        FILE = 'C:\CertBackup\CertTDE_MiApp.pvk',
        ENCRYPTION BY PASSWORD = 'OtraContraseñaSegura456!');

-- Habilitar TDE en la base de datos especifica
USE MiApp;
CREATE DATABASE ENCRYPTION KEY
    WITH ALGORITHM = AES_256
    ENCRYPTION BY SERVER CERTIFICATE CertTDE_MiApp;

ALTER DATABASE MiApp SET ENCRYPTION ON;
```

## Como se usa en un proyecto real

1. Se habilita TDE en bases de datos de produccion que almacenan datos sensibles (informacion personal de empleados/estudiantes en StaffTrainingPlatform), protegiendo contra robo fisico de backups o archivos de datos.
2. Se respalda el certificado y su clave privada inmediatamente despues de crearlo, en una ubicacion completamente separada de los backups de la base de datos (por ejemplo, un gestor de secretos corporativo).
3. Se combina TDE (proteccion en reposo) con `Encrypt=True` en la cadena de conexion (proteccion en transito, ver capitulo de .NET) para una estrategia de cifrado completa.
4. Se documenta el procedimiento de restauracion de un backup cifrado con TDE en otro servidor, que requiere primero restaurar el certificado antes de poder restaurar la base de datos.
5. Se prueba periodicamente que el certificado respaldado efectivamente permite restaurar un backup cifrado, dado que perderlo hace el backup completamente irrecuperable.

## Criterio junior

Un junior debe entender que protege TDE (archivos en reposo) y que no protege (datos en transito, usuarios autorizados con acceso activo).

## Criterio mid

Un mid debe habilitar TDE en una base de datos y respaldar correctamente el certificado necesario para restauracion futura.

## Criterio senior

Un senior debe diseñar la estrategia completa de gestion de claves de TDE, incluyendo respaldo seguro del certificado y procedimiento probado de restauracion en otro servidor.

## Errores comunes

- No respaldar el certificado y su clave privada, haciendo que un backup cifrado con TDE sea permanentemente irrecuperable ante perdida del servidor original.
- Guardar el respaldo del certificado en la misma ubicacion que los backups de la base de datos, perdiendo ambos ante el mismo incidente.
- Asumir que TDE protege datos en transito por la red, cuando en realidad solo protege archivos en reposo.
- No probar el procedimiento de restauracion de un backup cifrado con TDE en otro servidor antes de necesitarlo realmente en un incidente.

## Practica

- [ ] Habilita TDE en una base de datos de practica, siguiendo la jerarquia completa de claves.
- [ ] Respalda el certificado y su clave privada en una ubicacion separada.
- [ ] Prueba restaurar un backup cifrado con TDE en una instancia distinta, usando el certificado respaldado.
- [ ] Documenta que protege TDE y que no protege, distinguiendolo claramente de cifrado en transito.
- [ ] Diseña una estrategia de cifrado completa (TDE + Encrypt=True) para StaffTrainingPlatform.

## Preguntas de repaso

- Que protege TDE especificamente, y que no protege?
- Por que es critico respaldar el certificado por separado de los backups de la base de datos?
- Que jerarquia de claves usa TDE desde la Service Master Key hasta la DEK?
- Como se combina TDE con `Encrypt=True` en la cadena de conexion para una estrategia de cifrado completa?
