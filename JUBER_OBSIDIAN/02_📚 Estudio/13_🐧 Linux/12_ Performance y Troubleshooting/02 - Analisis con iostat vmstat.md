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

# 02 - Analisis con iostat vmstat

## Que vas a aprender

En este capitulo vas a aprender herramientas de analisis de performance mas profundas que las basicas ya vistas: **iostat** para I/O de disco detallado, y **vmstat** para memoria virtual y actividad general del sistema en una vista compacta.

## Explicacion

`iostat` (parte del paquete `sysstat`) muestra estadisticas detalladas de I/O por dispositivo de disco: **%util** (porcentaje de tiempo que el disco estuvo ocupado atendiendo requests, el indicador mas directo de saturacion de disco), **await** (tiempo promedio que una operacion espera para completarse, incluyendo tiempo en cola), y throughput de lectura/escritura. Un disco con %util cercano a 100% de forma sostenida es un cuello de botella real, independientemente de que CPU y memoria luzcan saludables.

`vmstat` da una vista compacta y actualizada periodicamente de multiples subsistemas simultaneamente: procesos en cola esperando CPU (`r`), memoria (incluyendo swap activo en las columnas `si`/`so`, cuya presencia sostenida es una señal seria de presion de memoria), y actividad de I/O. Es especialmente util como primer vistazo rapido antes de profundizar con herramientas mas especializadas en el subsistema especifico que resulte sospechoso.

## Modelo mental

Pensa `iostat` como un medidor especifico de trafico en la puerta de un deposito (el disco): te dice exactamente que tan ocupada esta esa puerta especifica y cuanto tarda cada camion en entrar y salir. `vmstat` es como un tablero general del edificio completo, mostrando de un vistazo el estado de varios sistemas distintos (CPU, memoria, I/O) simultaneamente, util para decidir rapidamente a que sistema especifico prestarle mas atencion despues.

## Ejemplo base

```bash
sudo apt install sysstat        # iostat y vmstat suelen venir en este paquete

iostat -x 2                      # estadisticas extendidas, actualizadas cada 2 segundos
# %util cercano a 100% de forma sostenida = disco saturado

vmstat 2 5                        # 5 muestras, cada 2 segundos
# columna "si"/"so" (swap in/out) activa de forma sostenida = presion real de memoria
# columna "r" alta y sostenida = procesos esperando CPU
```

## Como se usa en un proyecto real

1. Se usa `iostat -x` cuando se sospecha que el disco es el cuello de botella de un servidor lento, verificando `%util` y `await` en vez de adivinar.
2. Se usa `vmstat` como primer vistazo general para decidir rapidamente si el problema parece ser de CPU, memoria, o I/O, antes de profundizar con herramientas mas especificas.
3. Se monitorea `si`/`so` (swap activo) en `vmstat` como señal seria de presion de memoria real, mucho mas confiable que solo mirar "used" en `free`.
4. Se establecen umbrales de alerta basados en `%util` de disco sostenido y actividad de swap sostenida, integrando estas metricas al monitoreo continuo del sistema (ver capitulo de Monitoreo del libro de Docker para el equivalente a nivel de containers).
5. Se documenta un caso de diagnostico real usando estas herramientas como ejemplo de referencia para el equipo.

## Criterio junior

Un junior debe poder ejecutar `iostat -x` y `vmstat`, identificando las columnas mas importantes de cada una (`%util`, `si`/`so`).

## Criterio mid

Un mid debe diagnosticar si un servidor lento tiene su cuello de botella real en disco (via `iostat`) o en memoria/CPU (via `vmstat`), con evidencia concreta en vez de suposiciones.

## Criterio senior

Un senior debe integrar estas metricas en el monitoreo continuo de infraestructura, estableciendo umbrales de alerta correctos, y usarlas para justificar decisiones de dimensionamiento o migracion de almacenamiento (ver capitulo de Almacenamiento).

## Errores comunes

- No usar `iostat`/`vmstat` y quedarse solo con herramientas mas basicas, perdiendo visibilidad de la causa real de un problema de performance.
- Confundir un problema de disco lento con uno de memoria insuficiente, sin haber verificado con las herramientas correctas cual es el cuello de botella real.
- No establecer umbrales de alerta basados en estas metricas mas precisas, dependiendo solo de sintomas indirectos como "el servidor se siente lento".
- No documentar casos de diagnostico exitosos, perdiendo la oportunidad de construir un repositorio de conocimiento para el equipo.

## Practica

- [ ] Instala `sysstat` y ejecuta `iostat -x 2` en tu sistema, interpretando la columna `%util`.
- [ ] Ejecuta `vmstat 2 5` e interpreta las columnas `r`, `si`, `so`.
- [ ] Genera carga artificial en disco o memoria (con precaucion, en un entorno de practica) y observa el cambio en estas metricas.
- [ ] Diagnostica un caso simulado de "servidor lento" usando ambas herramientas para identificar la causa real.
- [ ] Documenta ese caso de diagnostico como referencia para tu equipo.

## Preguntas de repaso

- Que indica un `%util` cercano a 100% de forma sostenida en `iostat`?
- Por que la actividad sostenida en las columnas `si`/`so` de `vmstat` es una señal seria de presion de memoria?
- Como usarias `vmstat` como primer vistazo antes de profundizar con herramientas mas especificas?
- Que diferencia hay entre diagnosticar un problema de disco lento y uno de memoria insuficiente con estas herramientas?
