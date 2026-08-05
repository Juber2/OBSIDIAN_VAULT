---
tags:
  - Linux
  - Bash
  - Linux-Scripting
Creado: 2026-07-28
Relacionado:
  - Scripting con Bash
  - LINUX
---

# 05 - Automatizacion con Cron

## Que vas a aprender

En este capitulo vas a aprender a programar la ejecucion automatica y periodica de scripts con **cron**, la sintaxis de su expresion de programacion, y las consideraciones practicas para tareas programadas confiables (conectando con lo visto en el capitulo de Scheduling del libro de NestJS).

## Explicacion

**Cron** es el daemon estandar de Linux para ejecutar comandos en horarios programados, configurado via **crontab** (una tabla de tareas por usuario). La sintaxis de programacion tiene cinco campos: minuto, hora, dia del mes, mes, dia de la semana, cada uno pudiendo ser un valor especifico, un rango, una lista, o `*` (cualquier valor), seguidos del comando a ejecutar.

Consideraciones criticas para tareas de cron confiables: cron ejecuta con un **entorno minimo** (sin las variables de `.bashrc` normales), por lo que scripts que dependen de `PATH` personalizado u otras variables de entorno pueden fallar silenciosamente si no se configuran explicitamente dentro del script o crontab; se debe **redirigir la salida** a un log (`>> archivo.log 2>&1`) porque de lo contrario cron intenta enviarla por correo local, que en la mayoria de servidores modernos no esta configurado y la salida simplemente se pierde; y para tareas criticas, conviene agregar logica de **lock** (evitar que dos ejecuciones se superpongan si una tarda mas de lo esperado).

## Modelo mental

Pensa cron como un despertador confiable que suena exactamente a la hora programada, todos los dias, sin falta. Pero a diferencia de vos mismo ejecutando el comando manualmente (con todo tu entorno de shell personal ya configurado), cron despierta en una habitacion vacia y minimalista: si el comando necesita algo especifico de tu entorno personal, hay que dejarselo explicitamente preparado de antemano.

## Ejemplo base

```bash
crontab -e     # edita el crontab del usuario actual

# Formato: minuto hora dia-mes mes dia-semana comando
0 3 * * *  /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
# todos los dias a las 3:00 AM

*/15 * * * *  /opt/scripts/health-check.sh >> /var/log/health.log 2>&1
# cada 15 minutos

0 0 1 * *  /opt/scripts/reporte-mensual.sh >> /var/log/reporte.log 2>&1
# el primer dia de cada mes a medianoche

crontab -l     # lista las tareas programadas actuales
```

```bash
#!/bin/bash
# Dentro del script: PATH explicito, dado que cron no hereda el PATH interactivo normal
export PATH=/usr/local/bin:/usr/bin:/bin
```

## Como se usa en un proyecto real

1. Se redirige siempre la salida de tareas de cron a un archivo de log explicito, evitando depender de correo local que probablemente no esta configurado.
2. Se define `PATH` explicitamente al inicio de scripts ejecutados por cron, evitando fallos silenciosos por herramientas "no encontradas" que si funcionan en el shell interactivo normal.
3. Se usa `set -euo pipefail` (ver tema anterior) en scripts de cron para que fallos se detecten y logueen claramente, en vez de continuar en un estado inconsistente sin que nadie lo note.
4. Se agrega logica de lock (con `flock` o un archivo de lock simple) para tareas que no deben superponerse si una ejecucion tarda mas de lo esperado.
5. Se conecta este conocimiento con el capitulo de Scheduling del libro de NestJS: el mismo riesgo de "tareas duplicadas en multiples instancias" aplica igual a cron en servidores redundantes.

## Criterio junior

Un junior debe poder programar una tarea simple en crontab y entender la sintaxis basica de los cinco campos de tiempo.

## Criterio mid

Un mid debe redirigir correctamente la salida de tareas de cron a logs, y definir `PATH` explicito dentro de scripts para evitar fallos por entorno minimo.

## Criterio senior

Un senior debe diseñar tareas de cron robustas con manejo de errores, locks contra ejecucion duplicada, y logging confiable, ademas de coordinar la ejecucion de tareas criticas en infraestructura con multiples servidores redundantes.

## Errores comunes

- No redirigir la salida de una tarea de cron, perdiendo cualquier informacion de diagnostico cuando algo falla silenciosamente.
- Asumir que un script funciona igual bajo cron que cuando se ejecuta manualmente, sin considerar el entorno minimo (`PATH`, variables) que cron provee.
- No agregar manejo de errores robusto (`set -euo pipefail`) a scripts de cron, dejando que fallen silenciosamente sin que nadie se entere.
- No prevenir ejecuciones superpuestas de una tarea que ocasionalmente tarda mas de lo normal, generando comportamiento inconsistente.

## Practica

- [ ] Programa una tarea simple en tu crontab personal y verifica que se ejecuta en el horario esperado.
- [ ] Redirige la salida de esa tarea a un archivo de log y verifica su contenido.
- [ ] Define `PATH` explicito dentro de un script destinado a correr por cron.
- [ ] Agrega `set -euo pipefail` a un script de cron y simula un fallo para verificar que se registra correctamente.
- [ ] Investiga como usarias `flock` para evitar ejecuciones superpuestas de una tarea critica.

## Preguntas de repaso

- Que representan los cinco campos de tiempo en la sintaxis de crontab?
- Por que es importante redirigir explicitamente la salida de una tarea de cron a un log?
- Que diferencia de entorno hay entre ejecutar un script manualmente y ejecutarlo bajo cron?
- Que riesgo previene agregar logica de lock a una tarea de cron que puede tardar mas de lo esperado?
