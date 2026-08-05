---
tags:
  - Linux
  - Procesos
  - Linux-Procesos
Creado: 2026-07-28
Relacionado:
  - Procesos y Gestion del Sistema
  - LINUX
---

# 02 - ps top htop y Monitoreo de Procesos

## Que vas a aprender

En este capitulo vas a dominar las herramientas de monitoreo de procesos en tiempo real: `ps` para una foto puntual, `top` para monitoreo en vivo estandar, y `htop` como alternativa interactiva mas comoda, ademas de interpretar correctamente las metricas de CPU y memoria mostradas.

## Explicacion

`ps aux` (o `ps -ef`) da una **foto instantanea** del estado de todos los procesos en el momento exacto en que se ejecuta, util para scripts y capturas puntuales. `top` actualiza esa vista en tiempo real dentro de la misma terminal, mostrando los procesos ordenados por consumo (tipicamente CPU por defecto), permitiendo identificar rapidamente que esta consumiendo recursos ahora mismo. `htop` es una version mas moderna e interactiva de `top`, con colores, barras visuales de uso, y la posibilidad de interactuar directamente (matar procesos, cambiar orden) sin memorizar comandos.

Interpretar correctamente las metricas es clave: el porcentaje de CPU en `top` puede superar 100% en sistemas multi-nucleo (un proceso usando 200% significa que esta aprovechando dos nucleos completos), y distinguir memoria **RSS** (memoria fisica realmente usada) de **VSZ** (memoria virtual reservada, que puede ser mucho mayor sin estar realmente en uso) evita interpretaciones erroneas sobre cuanta memoria "realmente" consume un proceso.

## Modelo mental

Pensa `ps` como sacar una foto del trafico en un momento exacto, `top` como mirar una camara de trafico en vivo actualizandose cada segundo, y `htop` como esa misma camara pero con un panel de control interactivo donde podes hacer zoom, reordenar la vista, o directamente intervenir (matar un proceso) sin salir de la pantalla.

## Ejemplo base

```bash
ps aux --sort=-%cpu | head -10     # top 10 procesos por uso de CPU, foto puntual

top                                 # monitoreo en vivo (q para salir)
top -o %MEM                         # ordenado por uso de memoria

htop                                # version interactiva (F9 para matar un proceso seleccionado)

ps -eo pid,ppid,rss,vsz,comm --sort=-rss | head   # RSS vs VSZ explicitos
```

## Como se usa en un proyecto real

1. Se usa `top`/`htop` como primera herramienta de diagnostico cuando un servidor "se siente lento", identificando rapidamente el proceso responsable.
2. Se interpreta correctamente el porcentaje de CPU en sistemas multi-nucleo, evitando confusion cuando un proceso muestra mas de 100%.
3. Se distingue RSS de VSZ al diagnosticar problemas de memoria, dado que VSZ puede ser engañosamente alto sin representar consumo real.
4. Se usa `ps aux --sort` en scripts de monitoreo automatizado que necesitan una foto puntual, en vez de herramientas interactivas que no se prestan para automatizacion.
5. Se combina esto con los limites de recursos vistos en el libro de Docker (capitulo Senior), aplicando el mismo criterio de dimensionamiento basado en datos reales de consumo observado.

## Criterio junior

Un junior debe poder usar `ps aux`, `top` y `htop` para identificar que procesos consumen mas CPU o memoria en un momento dado.

## Criterio mid

Un mid debe interpretar correctamente porcentajes de CPU en sistemas multi-nucleo y distinguir RSS de VSZ al diagnosticar consumo de memoria.

## Criterio senior

Un senior debe diseñar scripts de monitoreo automatizado basados en `ps` para alertar sobre procesos anomalos, y usar esta informacion para dimensionar correctamente recursos de servidores e infraestructura.

## Errores comunes

- Interpretar un proceso al 150% de CPU como un error del sistema, sin entender que en multi-nucleo eso simplemente significa que usa un nucleo y medio.
- Confundir VSZ con el consumo real de memoria de un proceso, sacando conclusiones erroneas sobre uso de recursos.
- Depender solo de herramientas interactivas (`top`, `htop`) para monitoreo, sin tener una version automatizable (`ps` con opciones) para scripts.
- No revisar regularmente el estado de procesos en servidores de produccion, descubriendo problemas de recursos solo cuando ya causaron un incidente.

## Practica

- [ ] Identifica los procesos que mas CPU consumen en tu sistema con `ps aux --sort=-%cpu`.
- [ ] Explora `htop` interactivamente y practica ordenar por distintas metricas.
- [ ] Compara el RSS y VSZ de varios procesos y explica la diferencia observada.
- [ ] Escribe un comando `ps` que podrias usar en un script de monitoreo automatizado.
- [ ] Anota como interpretarias un proceso mostrando 300% de CPU en un servidor de 4 nucleos.

## Preguntas de repaso

- Que diferencia hay entre `ps` (foto puntual) y `top`/`htop` (monitoreo en vivo)?
- Por que el porcentaje de CPU de un proceso puede superar 100% en un sistema multi-nucleo?
- Que diferencia hay entre RSS y VSZ al medir el consumo de memoria de un proceso?
- Como usarias `ps` (en vez de `top`) dentro de un script de monitoreo automatizado?
