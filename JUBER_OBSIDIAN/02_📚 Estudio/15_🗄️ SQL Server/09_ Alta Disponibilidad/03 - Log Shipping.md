---
tags:
  - SQLServer
  - AltaDisponibilidad
  - SQLServer-AltaDisponibilidad
Creado: 2026-07-29
Relacionado:
  - Alta Disponibilidad
  - SQL SERVER
---

# 03 - Log Shipping

## Que vas a aprender

En este capitulo vas a aprender **Log Shipping**, la solucion de alta disponibilidad mas simple y antigua de SQL Server: envio periodico de backups de Transaction Log hacia uno o mas servidores secundarios, equivalente conceptual al log shipping tradicional que precedio a la streaming replication en Postgres.

## Explicacion

Log Shipping automatiza tres pasos en secuencia y de forma programada (via SQL Server Agent Jobs, ver capitulo de Administracion): **backup** periodico del Transaction Log en el servidor primario, **copia** del archivo de backup hacia el servidor secundario, y **restore** de ese log en la base de datos secundaria (que permanece en estado `RESTORING` o `STANDBY`, no completamente accesible como una replica de AG). A diferencia de Always On AG (replicacion casi en tiempo real, failover automatico posible), Log Shipping tiene un **delay inherente** determinado por la frecuencia configurada (tipicamente cada 15 minutos o mas), y el **failover es manual**, requiriendo intervencion humana para promover el secundario a primario.

Por su simplicidad y menor requerimiento de infraestructura (no necesita cluster subyacente como AG o FCI), Log Shipping sigue siendo relevante para escenarios de disaster recovery donde un RPO de minutos es aceptable y no se justifica la complejidad operativa de Always On AG.

## Modelo mental

Pensa Log Shipping como enviar por correo postal fotocopias periodicas de un libro de contabilidad a una sucursal remota: no es instantaneo (hay un delay entre cada envio), y si la oficina principal se incendia, alguien en la sucursal remota tiene que decidir manualmente "a partir de ahora, esta es la copia oficial" (el failover manual), a diferencia de Always On AG que es mas parecido a una transmision en vivo con conmutacion automatica.

## Ejemplo base

```sql
-- Log Shipping se configura principalmente por SSMS o stored procedures de sistema.
-- Configuracion simplificada del lado del backup:

EXEC master.dbo.sp_add_log_shipping_primary_database
    @database = 'MiApp',
    @backup_directory = 'C:\LogShipping\Backup',
    @backup_share = '\\SQLNODE1\LogShipping',
    @backup_job_name = 'LSBackup_MiApp',
    @backup_retention_period = 4320;

-- Verificar el estado de Log Shipping
SELECT * FROM msdb.dbo.log_shipping_monitor_secondary;
```

## Como se usa en un proyecto real

1. Se elige Log Shipping para disaster recovery hacia un sitio remoto cuando un delay de algunos minutos (RPO) es aceptable y no se justifica la complejidad de Always On AG.
2. Se documenta el procedimiento manual de failover (promover el secundario, redirigir aplicaciones) dado que, a diferencia de AG, no hay failover automatico.
3. Se monitorea regularmente el estado de sincronizacion (`log_shipping_monitor_secondary`) para detectar si el secundario se esta atrasando mas de lo esperado.
4. Se conecta esto con el log shipping tradicional que historicamente precedio a streaming replication en Postgres, entendiendo la evolucion similar de ambos ecosistemas hacia replicacion mas cercana a tiempo real.
5. Se considera Log Shipping como una alternativa mas simple y economica frente a Always On AG cuando los requisitos de RPO/RTO lo permiten.

## Criterio junior

Un junior debe entender los tres pasos de Log Shipping (backup, copia, restore) y que el failover es manual.

## Criterio mid

Un mid debe configurar Log Shipping y monitorear su estado de sincronizacion regularmente.

## Criterio senior

Un senior debe decidir entre Log Shipping, Always On AG, o FCI segun los requisitos reales de RPO/RTO y la complejidad operativa que el equipo puede sostener.

## Errores comunes

- Elegir Log Shipping para un escenario que realmente necesita failover automatico (RTO muy bajo), donde Always On AG seria mas apropiado.
- No documentar el procedimiento manual de failover, dejando al equipo sin un plan claro durante un incidente real.
- No monitorear regularmente el estado de sincronizacion, descubriendo que el secundario esta muy atrasado solo durante un incidente.
- Sobrestimar la complejidad de Always On AG cuando en realidad el sistema necesita RPO/RTO bajo y Log Shipping no es suficiente.

## Practica

- [ ] Investiga la configuracion de Log Shipping (backup, copia, restore) paso a paso.
- [ ] Documenta un procedimiento manual de failover para un escenario de Log Shipping.
- [ ] Consulta las vistas de monitoreo de Log Shipping para verificar el estado de sincronizacion.
- [ ] Compara Log Shipping, Always On AG, y FCI en una tabla de RPO/RTO/complejidad.
- [ ] Decide que solucion usarias para un sistema de produccion hipotetico con RPO de 1 hora aceptable.

## Preguntas de repaso

- Cuales son los tres pasos que automatiza Log Shipping?
- Por que el failover en Log Shipping es manual y no automatico?
- En que escenario elegirias Log Shipping en vez de Always On AG?
- Como se relaciona esta tecnica con la evolucion historica de replicacion en Postgres?
