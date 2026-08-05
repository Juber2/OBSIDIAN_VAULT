---
tags:
  - Linux
  - Performance
  - Linux-Performance
Creado: 2026-07-28
Relacionado:
  - Performance y Troubleshooting
  - LINUX
---

# 01 - Monitoreo de CPU Memoria y Disco

## Que vas a aprender

En este capitulo vas a profundizar en el monitoreo de recursos del sistema mas alla de `top`/`htop` (ya vistos): el **load average**, la diferencia entre memoria **usada** y **disponible** (frecuentemente malinterpretada), y el uso real de espacio en disco.

## Explicacion

El **load average** (mostrado por `uptime`, tres numeros: promedio de 1, 5 y 15 minutos) representa cuantos procesos, en promedio, estaban esperando CPU disponible en ese periodo. Un load average de 4 en un sistema de 4 nucleos significa que la CPU esta aproximadamente al limite de su capacidad; el mismo valor en un sistema de 16 nucleos indica mucha capacidad ociosa disponible. Interpretar el load average sin conocer el numero de nucleos disponibles (`nproc`) lleva a conclusiones erroneas.

Sobre memoria: Linux usa agresivamente RAM libre para **cache de disco**, lo cual es beneficioso para performance (acelera lecturas repetidas) pero hace que herramientas simples muestren "poca memoria libre" cuando en realidad esa memoria esta disponible para reclamarse instantaneamente si una aplicacion la necesita. La columna correcta a mirar en `free -h` es **available**, no **free**, para saber cuanta memoria realmente puede usar una aplicacion nueva sin generar swapping.

## Modelo mental

Pensa el load average como la fila de espera en una caja de supermercado: el numero de personas en fila (procesos esperando) tiene sentido solo si sabes cuantas cajas (nucleos) estan atendiendo. Pensa la memoria "usada por cache" como libros que la biblioteca dejo temporalmente sobre una mesa disponible porque nadie mas los necesitaba en ese momento: parece "ocupado", pero apenas alguien necesita esa mesa para algo mas importante, los libros se guardan instantaneamente sin perder nada.

## Ejemplo base

```bash
uptime                      # load average de 1, 5 y 15 minutos
nproc                        # numero de nucleos de CPU disponibles

free -h                      # memoria: fijate en "available", no en "free"
#               total   used   free  shared  buff/cache  available
# Mem:            16Gi   4.0Gi  1.0Gi   200Mi       11Gi        11Gi

df -h                         # espacio en disco por filesystem montado
du -sh /var/log/*             # que subdirectorios ocupan mas espacio dentro de uno especifico
```

## Como se usa en un proyecto real

1. Se interpreta el load average siempre en relacion al numero de nucleos disponibles (`nproc`), nunca como un numero absoluto aislado sin ese contexto.
2. Se mira "available" en `free -h`, no "free", para evaluar correctamente si un sistema esta bajo presion real de memoria.
3. Se combina `df -h` (espacio por filesystem) con `du -sh` (que directorio especifico esta consumiendo ese espacio) para diagnosticar rapidamente un disco lleno.
4. Se establecen umbrales de alerta basados en estas metricas correctamente interpretadas, evitando falsas alarmas por malinterpretar memoria cacheada como memoria realmente agotada.
5. Se conecta este conocimiento con las metricas de containers vistas en el capitulo de Performance del libro de Docker, dado que los mismos conceptos (RSS, cache, load) aplican tanto al host completo como a un container individual.

## Criterio junior

Un junior debe poder leer `uptime`, `free -h` y `df -h`, entendiendo la columna "available" como la correcta para memoria realmente disponible.

## Criterio mid

Un mid debe interpretar el load average en relacion al numero de nucleos, y diagnosticar un disco lleno combinando `df` con `du` para encontrar la causa especifica.

## Criterio senior

Un senior debe establecer umbrales de alerta correctos basados en metricas bien interpretadas, evitando falsas alarmas comunes por malinterpretar memoria cacheada, y diseñar dashboards de monitoreo que muestren las metricas correctas de forma clara para el equipo.

## Errores comunes

- Interpretar un load average alto como problema critico sin considerar cuantos nucleos tiene el sistema.
- Alarmarse por poca memoria "free" sin revisar "available", generando falsas alertas por no entender el cache de disco de Linux.
- No combinar `df` con `du` al diagnosticar un disco lleno, sin encontrar la causa raiz especifica.
- No establecer umbrales de alerta correctamente calibrados, generando ruido de alertas que el equipo termina ignorando.

## Practica

- [ ] Revisa el load average de tu sistema y comparalo con el numero de nucleos disponibles.
- [ ] Revisa `free -h` distinguiendo "free" de "available" y explicando la diferencia.
- [ ] Diagnostica que esta ocupando espacio en un directorio especifico combinando `df` y `du`.
- [ ] Establece un umbral de alerta de memoria basado en "available", no en "free".
- [ ] Anota como conecta este conocimiento con las metricas de containers vistas en el libro de Docker.

## Preguntas de repaso

- Por que un load average de 4 significa cosas distintas en un sistema de 4 nucleos versus uno de 16?
- Por que la columna "available" de `free -h` es mas confiable que "free" para evaluar presion real de memoria?
- Como combinarias `df` y `du` para diagnosticar la causa de un disco lleno?
- Como se relacionan estas metricas con las vistas en el capitulo de Performance del libro de Docker?
