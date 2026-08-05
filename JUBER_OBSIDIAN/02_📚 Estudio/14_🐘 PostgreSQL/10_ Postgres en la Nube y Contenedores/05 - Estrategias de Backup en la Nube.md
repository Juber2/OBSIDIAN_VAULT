---
tags:
  - PostgreSQL
  - Cloud
  - Postgres-Cloud
Creado: 2026-07-29
Relacionado:
  - Postgres en la Nube y Contenedores
  - POSTGRESQL
---

# 05 - Estrategias de Backup en la Nube

## Que vas a cerrar

En este capitulo vas a cerrar el capitulo consolidando estrategias de backup especificas de la nube: backups automaticos de servicios gestionados, envio de `pg_dump`/WAL hacia almacenamiento de objetos (S3 y equivalentes), y la aplicacion de la regla 3-2-1 ya vista en el libro de Linux especificamente a Postgres en la nube.

## Explicacion

Los servicios gestionados (ver tema de RDS/Cloud SQL) ofrecen backups automaticos con retencion configurable y, en muchos casos, PITR integrado sin necesitar configurar manualmente el archivado de WAL (ya visto en el capitulo de Administracion). Aun asi, la regla 3-2-1 (ya vista en el libro de Linux: al menos 3 copias, 2 tipos de medio, 1 fuera del sitio) sigue aplicando: depender **unicamente** de los backups automaticos de un unico proveedor cloud es depender de un unico "sitio", sin proteccion real ante un problema con la cuenta de ese proveedor especifico (suspension accidental, error de configuracion, o el escenario extremo de un incidente de seguridad en la cuenta).

Para Postgres auto-gestionado (en Docker o Kubernetes, ya vistos en este capitulo), la estrategia tipica es enviar backups de `pg_dump` (o el archivado continuo de WAL para PITR) hacia almacenamiento de objetos (S3, Google Cloud Storage, Azure Blob Storage), con políticas de ciclo de vida que muevan backups viejos a almacenamiento mas economico o los eliminen segun la politica de retencion definida.

## Modelo mental

Pensa depender solo del backup automatico de un proveedor cloud como guardar todas tus fotos importantes unicamente en la nube de un unico servicio, confiando en que ese servicio nunca falle ni tenga un problema con tu cuenta especifica. La regla 3-2-1 aplicada a la nube es como ademas tener una copia en un disco fisico separado y otra en un segundo servicio de nube distinto: cualquier problema con un unico proveedor, por mas confiable que sea normalmente, no te deja sin ninguna copia recuperable.

## Ejemplo base

```bash
# Backup automatizado enviado a almacenamiento de objetos (ejemplo conceptual con AWS CLI)
pg_dump -Fc mi_app | gzip > backup-$(date +%Y%m%d).dump.gz
aws s3 cp backup-$(date +%Y%m%d).dump.gz s3://mi-bucket-backups/postgres/

# Politica de ciclo de vida del bucket: mover a almacenamiento economico despues de 30 dias,
# eliminar despues de 365 dias (configurado en la consola o infraestructura como codigo)
```

```bash
# Verificar que los backups automaticos de un servicio gestionado (ejemplo RDS) estan activos
aws rds describe-db-instances --db-instance-identifier mi-instancia \
  --query 'DBInstances[0].BackupRetentionPeriod'
```

## Como se usa en un proyecto real

1. Se activan y verifican los backups automaticos de cualquier servicio gestionado usado, confirmando el periodo de retencion configurado.
2. Se complementa con al menos un backup adicional independiente (exportado periodicamente hacia otro proveedor o almacenamiento) para no depender de un unico proveedor, aplicando la regla 3-2-1.
3. Se envian backups de Postgres auto-gestionado hacia almacenamiento de objetos con politicas de ciclo de vida automatizadas, evitando gestion manual de retencion.
4. Se prueba la restauracion desde el backup en la nube regularmente, aplicando el mismo principio de "probar la restauracion" ya insistido en el libro de Linux.
5. Se documenta el RPO/RTO real que la estrategia de backup en la nube puede garantizar, comunicandolo claramente al negocio.

## Criterio junior

Un junior debe poder verificar que backups automaticos estan activos en un servicio gestionado, y enviar un `pg_dump` manual hacia almacenamiento de objetos.

## Criterio mid

Un mid debe automatizar el envio de backups hacia almacenamiento de objetos con politicas de ciclo de vida, y aplicar la regla 3-2-1 en un contexto cloud.

## Criterio senior

Un senior debe diseñar la estrategia completa de backup y disaster recovery de bases de datos en la nube, garantizando independencia de un unico proveedor y probando restauraciones regularmente.

## Errores comunes

- Depender unicamente de los backups automaticos de un unico proveedor cloud, sin ninguna copia independiente fuera de esa cuenta especifica.
- No verificar activamente que los backups automaticos de un servicio gestionado realmente estan configurados y funcionando, asumiendo que "vienen incluidos por defecto" sin confirmarlo.
- No aplicar politicas de ciclo de vida a backups en almacenamiento de objetos, acumulando costo indefinidamente sin control.
- No probar restauraciones desde backups en la nube, descubriendo problemas de acceso o formato solo durante un incidente real.

## Practica

- [ ] Verifica la configuracion de backups automaticos de un servicio gestionado (real o de documentacion).
- [ ] Automatiza el envio de un `pg_dump` hacia almacenamiento de objetos (S3 o equivalente).
- [ ] Configura (o investiga como configurarias) una politica de ciclo de vida para esos backups.
- [ ] Aplica la regla 3-2-1 diseñando una estrategia de backup completa para un proyecto cloud real.
- [ ] Prueba restaurar un backup descargado desde almacenamiento de objetos.

## Preguntas de repaso

- Por que depender unicamente de los backups automaticos de un unico proveedor cloud viola la regla 3-2-1?
- Que verificarias para confirmar que los backups automaticos de un servicio gestionado realmente funcionan?
- Que ventaja da una politica de ciclo de vida en almacenamiento de objetos para backups?
- Como aplicarias el principio de "probar la restauracion" a un backup almacenado en la nube?
