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

# 05 - Automatizacion de Tareas Administrativas

## Que vas a aprender

En este capitulo vas a integrar todo el capitulo en la practica de **automatizar tareas administrativas recurrentes**: scripts de mantenimiento combinando lo visto en Scripting con Bash, cron para programacion, y el criterio de cuando automatizar versus cuando una tarea manual sigue siendo apropiada.

## Explicacion

Un administrador de sistemas efectivo automatiza tareas repetitivas para reducir tanto el tiempo invertido como el riesgo de error humano en procesos manuales propensos a olvidos. Candidatos tipicos de automatizacion: rotacion y limpieza de logs viejos, backups periodicos (ya visto), verificacion de salud de servicios criticos con alertas ante fallos, limpieza de archivos temporales, y renovacion automatica de certificados SSL.

El criterio para decidir que automatizar sigue una regla simple: si una tarea se repite regularmente y sigue pasos predecibles, es candidata a automatizacion. Si una tarea requiere juicio humano caso por caso (decidir si un pico de trafico es un ataque o una promocion exitosa), automatizarla completamente puede ser contraproducente; en esos casos, la automatizacion util es la que **alerta** a un humano con contexto suficiente para decidir, no la que actua de forma completamente autonoma sin supervision.

## Modelo mental

Pensa la automatizacion administrativa como delegar tareas rutinarias a un asistente confiable: las tareas mecanicas y predecibles (limpiar, respaldar, verificar) se le pueden confiar completamente sin supervision constante. Las decisiones que requieren criterio (¿este comportamiento raro es un problema real o normal?) siguen necesitando que el asistente te avise con la informacion relevante, en vez de decidir y actuar completamente solo.

## Ejemplo base

```bash
#!/bin/bash
# Script de mantenimiento diario, combinando practicas de capitulos anteriores
set -euo pipefail

LOG="/var/log/mantenimiento.log"

log() { echo "[$(date +%Y-%m-%d\ %H:%M:%S)] $1" >> "$LOG"; }

log "Iniciando mantenimiento diario"

find /tmp -type f -mtime +7 -delete && log "Limpieza de /tmp completada"

for servicio in nginx postgresql; do
    if ! systemctl is-active --quiet "$servicio"; then
        log "ALERTA: $servicio esta caido"
        # Aca se integraria con un sistema de alertas real (email, Slack, etc.)
    fi
done

df -h / | awk 'NR==2 {if ($5+0 > 85) print "ALERTA: disco raiz al " $5}' >> "$LOG"

log "Mantenimiento diario completado"
```

```bash
# Crontab: ejecuta el script de mantenimiento todos los dias
0 4 * * *  /opt/scripts/mantenimiento-diario.sh
```

## Como se usa en un proyecto real

1. Se automatizan tareas mecanicas y predecibles (limpieza, backup, verificacion de salud) combinando todo lo visto en el capitulo de Scripting con Bash y cron.
2. Se integra la automatizacion con un sistema de alertas real (no solo logging pasivo) para que un humano se entere activamente de problemas que requieren su atencion.
3. Se documenta cada script de automatizacion administrativa (proposito, frecuencia, que hacer si falla) como parte del conocimiento operativo del equipo.
4. Se revisa periodicamente que scripts de automatizacion siguen siendo relevantes, eliminando los que ya no aplican a la infraestructura actual.
5. Se conecta esta automatizacion con el criterio de decisiones arquitectonicas ya visto en el capitulo Senior del libro de Docker: automatizar no es "siempre mejor" por defecto, se justifica cuando realmente ahorra tiempo/riesgo real.

## Criterio junior

Un junior debe poder escribir un script de mantenimiento simple (limpieza de temporales, verificacion basica) y programarlo con cron.

## Criterio mid

Un mid debe integrar alertas reales (no solo logs pasivos) en scripts de automatizacion, y decidir que tareas son buenas candidatas a automatizar versus cuales requieren supervision humana.

## Criterio senior

Un senior debe diseñar la suite completa de automatizacion administrativa de la infraestructura, estableciendo el balance correcto entre accion autonoma y alertas que requieren decision humana, y documentando todo el conocimiento operativo resultante.

## Errores comunes

- Automatizar completamente decisiones que en realidad requieren juicio humano, causando acciones incorrectas ante situaciones ambiguas.
- No integrar alertas reales, dejando que un script detecte un problema pero solo lo registre en un log que nadie revisa activamente.
- No documentar scripts de automatizacion, generando "magia negra" operativa que solo una persona entiende completamente.
- No revisar periodicamente la relevancia de scripts existentes, manteniendo automatizacion obsoleta que ya no aplica a la infraestructura actual.

## Practica

- [ ] Escribe un script de mantenimiento que combine limpieza, verificacion de servicios, y chequeo de espacio en disco.
- [ ] Integra ese script con logging robusto siguiendo lo visto en el capitulo de Scripting.
- [ ] Programa el script con cron para ejecucion diaria.
- [ ] Diseña como integrarias una alerta real (email, webhook) ante un problema detectado.
- [ ] Documenta el proposito y comportamiento esperado de ese script para tu equipo.

## Preguntas de repaso

- Que tipo de tareas son buenas candidatas para automatizacion completa versus cuales requieren alertar a un humano?
- Por que un script que solo registra un problema en un log pasivo puede no ser suficiente?
- Que documentacion deberia acompañar a cualquier script de automatizacion administrativa?
- Como decidirias si un script de automatizacion existente sigue siendo relevante con el tiempo?
