---
tags:
  - Linux
  - Administracion
  - Linux-Servidores
Creado: 2026-07-28
Relacionado:
  - Administracion de Servidores
  - LINUX
---

# 03 - Backups y Recuperacion

## Que vas a aprender

En este capitulo vas a consolidar una estrategia de **backup** completa a nivel de sistema Linux: `rsync` para sincronizacion eficiente, `tar` para archivado, y el diseño de una politica de backup completa (conectando con lo ya visto en el libro de Docker sobre backup de volumenes).

## Explicacion

`rsync` sincroniza archivos de forma eficiente, transfiriendo solo las diferencias entre origen y destino (no copiando todo de nuevo cada vez), ideal para backups incrementales rapidos, especialmente sobre red (`rsync -avz origen usuario@servidor:destino`). `tar` empaqueta (y opcionalmente comprime) un conjunto de archivos en un unico archivo, util para snapshots puntuales completos o para transferir un conjunto de datos como una unidad.

Una politica de backup real y completa sigue la regla **3-2-1**: al menos 3 copias de los datos, en 2 tipos de medio distintos, con 1 copia fuera del sitio (offsite). Esto conecta directamente con lo ya visto en el capitulo de backup de volumenes del libro de Docker: el mismo principio (backup automatizado, probado regularmente, almacenado externamente) aplica igual a nivel de sistema completo que a nivel de un volumen especifico de container.

## Modelo mental

Pensa `rsync` como un mensajero eficiente que, en cada viaje, solo lleva lo que cambio desde la ultima vez que visito ese destino, en vez de cargar todo el contenido completo cada vez. `tar` es como empaquetar todo en una caja sellada para archivar o transportar como una unica unidad. La regla 3-2-1 es como no confiar toda tu informacion importante a un unico lugar: si la casa se incendia (falla el servidor), necesitas una copia guardada en otro lugar completamente distinto.

## Ejemplo base

```bash
# rsync: backup incremental eficiente, local o remoto
rsync -avz --delete /var/www/ usuario@servidor-backup:/backups/www/

# tar: archivo comprimido puntual completo
tar czf backup-$(date +%Y%m%d).tar.gz /etc /var/www

# Script simple de backup con rotacion (mantener solo los ultimos 7)
find /backups -name "backup-*.tar.gz" -mtime +7 -delete
```

## Como se usa en un proyecto real

1. Se automatiza el backup con `rsync` o `tar` como tarea de cron (ver capitulo de Scripting con Bash), con logging y manejo de errores robusto.
2. Se aplica la regla 3-2-1: copias locales para recuperacion rapida, y al menos una copia offsite (otro servidor, almacenamiento cloud) para proteger contra la perdida total del servidor principal.
3. Se prueba el proceso de restauracion regularmente, no solo el de backup, siguiendo exactamente el mismo principio ya visto en el capitulo de Backup de Volumenes del libro de Docker.
4. Se implementa rotacion de backups viejos para no acumular espacio de almacenamiento indefinidamente.
5. Se documenta claramente el RPO (cuanto dato se puede permitir perder) y RTO (cuanto tiempo de recuperacion es aceptable) para cada sistema, guiando la frecuencia y metodo de backup elegido.

## Criterio junior

Un junior debe poder hacer un backup basico con `rsync` o `tar`, y entender la diferencia entre ambos.

## Criterio mid

Un mid debe automatizar backups con cron, implementar rotacion de backups viejos, y probar el proceso de restauracion regularmente.

## Criterio senior

Un senior debe diseñar la estrategia completa de backup y disaster recovery de la infraestructura aplicando la regla 3-2-1, definiendo RPO/RTO por sistema, y coordinando con las estrategias de backup de volumenes de containers ya vistas en el libro de Docker.

## Errores comunes

- Hacer backup solo localmente en el mismo servidor, perdiendo toda proteccion si ese servidor completo falla o es comprometido.
- No probar el proceso de restauracion, descubriendo recien durante un incidente real que el backup estaba corrupto o incompleto.
- No implementar rotacion de backups viejos, acumulando espacio de almacenamiento indefinidamente sin control.
- No documentar RPO/RTO, dejando la frecuencia de backup como una decision arbitraria sin justificacion clara del riesgo aceptado.

## Practica

- [ ] Realiza un backup de un directorio de practica usando `rsync`.
- [ ] Realiza un backup comprimido puntual con `tar`.
- [ ] Automatiza uno de estos backups como tarea de cron con logging.
- [ ] Implementa rotacion simple que elimine backups mas viejos que N dias.
- [ ] Prueba restaurar desde uno de tus backups y verifica que los datos son correctos.

## Preguntas de repaso

- Que ventaja tiene `rsync` sobre `tar` para backups incrementales frecuentes?
- Que significa la regla 3-2-1 de backup?
- Por que es tan importante probar el proceso de restauracion, no solo el de backup?
- Como definirias el RPO y RTO apropiados para un sistema critico de tu infraestructura?
