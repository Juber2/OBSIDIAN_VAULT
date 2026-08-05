---
tags:
  - Linux
  - Proyectos
  - Linux-Proyectos
Creado: 2026-07-28
Relacionado:
  - Proyectos
  - LINUX
---

# 02 - Proyecto Script de Backup Automatizado

## Que vas a construir

Un sistema de backup completo y automatizado para el servidor del proyecto anterior, aplicando los capitulos de Scripting con Bash, Almacenamiento, y Administracion de Servidores.

## Alcance funcional

- Script de backup robusto (con `set -euo pipefail`, logging, y manejo de errores) que respalde archivos de aplicacion y base de datos.
- Backup programado automaticamente via cron, con salida redirigida a un log.
- Rotacion automatica que elimina backups mas viejos que un umbral definido.
- Backup enviado a un destino externo (otro servidor, o almacenamiento remoto), no solo local.
- Proceso de restauracion documentado y probado al menos una vez.

## Checklist tecnica

- [ ] El script usa `set -euo pipefail` y maneja errores explicitamente donde corresponde.
- [ ] La tarea de cron redirige correctamente stdout y stderr a un archivo de log.
- [ ] El script define `PATH` explicito, sin depender del entorno interactivo normal.
- [ ] Los backups viejos se eliminan automaticamente segun una politica de retencion definida.
- [ ] Se realizo al menos una restauracion completa de prueba, verificando que los datos son correctos.

## Criterio de exito

El proyecto se considera completo cuando el backup corre automaticamente sin intervencion manual, genera logs claros de cada ejecucion, mantiene solo la cantidad de backups definida por la politica de retencion, y se demostro (con una restauracion de prueba real) que los backups son efectivamente recuperables.

## Errores comunes a evitar

- Backups que solo se guardan localmente, sin ninguna copia fuera del servidor original.
- No probar nunca el proceso de restauracion, confiando ciegamente en que el backup "deberia funcionar".
- Scripts de cron sin logging, perdiendo visibilidad de si el backup realmente se ejecuto correctamente cada noche.

## Extension opcional

Agregar una alerta (email o webhook) que notifique si el script de backup falla, en vez de descubrirlo recien cuando se necesita restaurar.
