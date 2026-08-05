---
tags:
  - SQLServer
  - Nube
  - SQLServer-Nube
Creado: 2026-07-29
Relacionado:
  - SQL Server en la Nube y Contenedores
  - SQL SERVER
---

# 05 - Backup a Azure Storage

## Que vas a cerrar

En este capitulo vas a cerrar el capitulo aprendiendo a respaldar SQL Server directamente hacia **Azure Blob Storage**, la practica moderna recomendada frente a guardar backups solo en disco local, aplicando el mismo principio 3-2-1 de backups ya insistido en el libro de Linux.

## Explicacion

SQL Server soporta `BACKUP DATABASE ... TO URL`, escribiendo el archivo de backup directamente hacia un contenedor de Azure Blob Storage en vez de un disco local, eliminando el paso manual de copiar el archivo despues de generarlo. Esto es particularmente relevante para SQL Server corriendo en contenedores o Kubernetes (donde el almacenamiento local del Pod puede ser efimero) y para cumplir la regla de tener copias en una ubicacion geografica distinta a la del servidor de produccion, el mismo principio 3-2-1 (tres copias, dos medios distintos, una copia fuera del sitio) ya cubierto en profundidad en el libro de Linux.

Azure SQL Database (el servicio administrado) ya incluye backups automaticos con PITR gestionado por Microsoft sin configuracion adicional; este tema aplica principalmente a instancias de SQL Server autogestionadas (en VM, contenedor, o Kubernetes) donde la responsabilidad del backup recae en el equipo.

## Modelo mental

Pensa `BACKUP DATABASE ... TO URL` como enviar el backup directamente a una caja fuerte en otro edificio en el momento en que se genera, en vez de dejarlo primero en un cajon local (disco del servidor) y despues acordarte de llevarlo manualmente a la caja fuerte despues. Elimina el paso manual y el riesgo de olvidarlo, aplicando el mismo principio de "una copia fuera del sitio" ya visto con la regla 3-2-1 en el libro de Linux.

## Ejemplo base

```sql
-- Crear credencial para acceder a Azure Blob Storage
CREATE CREDENTIAL [https://miappstorage.blob.core.windows.net/backups]
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
    SECRET = 'sv=2022-11-02&ss=b&srt=co&sp=rwdlac...';

-- Backup directo hacia Azure Blob Storage
BACKUP DATABASE MiApp
    TO URL = 'https://miappstorage.blob.core.windows.net/backups/MiApp_Full.bak'
    WITH COMPRESSION, CHECKSUM;
```

## Como se usa en un proyecto real

1. Se configura `BACKUP DATABASE ... TO URL` para instancias autogestionadas de SQL Server (contenedor, Kubernetes, VM), asegurando que los backups existan fuera del almacenamiento local efimero.
2. Se automatiza este backup como un SQL Server Agent Job (ver capitulo de Administracion), combinando la estrategia Full/Differential/Log ya estudiada con el destino en la nube.
3. Se aplica la regla 3-2-1 completa: backup local para restauracion rapida, mas backup en Azure Storage para recuperacion ante desastre geografico, el mismo principio ya insistido en el libro de Linux.
4. Se confirma que Azure SQL Database (el servicio administrado) ya gestiona esto automaticamente, reservando este tema especificamente para instancias autogestionadas.
5. Se prueba regularmente la restauracion desde Azure Storage, no solo se confia en que el backup "esta ahi" sin verificarlo.

## Criterio junior

Un junior debe poder ejecutar un `BACKUP DATABASE ... TO URL` basico hacia Azure Blob Storage.

## Criterio mid

Un mid debe automatizar este backup como un Agent Job y aplicar la estrategia Full/Differential/Log hacia el destino en la nube.

## Criterio senior

Un senior debe diseñar la estrategia completa de backup 3-2-1 para sistemas autogestionados, y garantizar pruebas regulares de restauracion desde el destino remoto.

## Errores comunes

- Guardar backups solo en el almacenamiento local del contenedor o VM, perdiendolos si esa infraestructura se destruye.
- No usar `CHECKSUM` en el backup, perdiendo la capacidad de detectar corrupcion del archivo de backup mismo.
- Confundir esta necesidad con Azure SQL Database administrado, que ya gestiona backups automaticamente sin este paso manual.
- No probar la restauracion desde Azure Storage regularmente, descubriendo problemas de acceso o formato solo durante un incidente real.

## Practica

- [ ] Configura una credencial de Azure Blob Storage para SQL Server.
- [ ] Ejecuta un backup directo hacia Azure Storage con `BACKUP DATABASE ... TO URL`.
- [ ] Automatiza este backup como un Agent Job programado.
- [ ] Practica una restauracion completa desde el backup guardado en Azure Storage.
- [ ] Diseña la estrategia 3-2-1 completa para una instancia autogestionada de SQL Server.

## Preguntas de repaso

- Que ventaja tiene `BACKUP DATABASE ... TO URL` frente a respaldar a disco local y copiar despues manualmente?
- Por que este tema aplica principalmente a instancias autogestionadas y no a Azure SQL Database administrado?
- Como se relaciona esto con la regla 3-2-1 de backups ya vista en el libro de Linux?
- Que verificarias antes de confiar en que un backup en Azure Storage es realmente restaurable?

## Cierre del capitulo

Con este tema cierras el capitulo de SQL Server en la nube y contenedores, habiendo cubierto Docker para desarrollo local, Azure SQL Database como servicio administrado, Kubernetes para despliegues autogestionados con alta disponibilidad, migraciones EF Core seguras contra la nube, y backups resilientes hacia almacenamiento remoto.
