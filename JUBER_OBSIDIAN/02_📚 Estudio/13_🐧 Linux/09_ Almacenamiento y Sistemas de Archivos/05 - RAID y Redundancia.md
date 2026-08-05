---
tags:
  - Linux
  - Almacenamiento
  - Linux-Almacenamiento
Creado: 2026-07-28
Relacionado:
  - Almacenamiento y Sistemas de Archivos
  - LINUX
---

# 05 - RAID y Redundancia

## Que vas a aprender

En este capitulo vas a aprender los niveles de **RAID** mas comunes (0, 1, 5, 10) para combinar multiples discos fisicos, entendiendo el tradeoff entre performance, redundancia, y espacio util disponible.

## Explicacion

**RAID 0** (striping) distribuye datos entre discos para mejorar performance, pero sin redundancia: si un disco falla, se pierden todos los datos, siendo en realidad **mas riesgoso** que un unico disco, no mas seguro. **RAID 1** (mirroring) duplica los mismos datos en dos discos: si uno falla, el otro sigue funcionando sin perdida de datos, a costa de usar solo el 50% del espacio total disponible. **RAID 5** distribuye datos con paridad calculada entre 3+ discos, tolerando el fallo de un unico disco con mejor eficiencia de espacio que RAID 1. **RAID 10** combina mirroring y striping (requiere minimo 4 discos), dando buena performance y redundancia simultaneamente, a costa de mayor cantidad de discos necesarios.

Es critico entender que **RAID no es un reemplazo de backup**: protege contra la falla fisica de un disco, pero no contra errores humanos (borrar un archivo por error), corrupcion de datos a nivel de aplicacion, o ransomware, todo lo cual se replica igual de "fielmente" en todos los discos del arreglo RAID.

## Modelo mental

Pensa RAID 0 como escribir la mitad de un documento importante en cada una de dos hojas distintas: mas rapido de escribir en paralelo, pero si perdes cualquiera de las dos hojas, el documento completo queda inutilizable. RAID 1 es como tener dos copias identicas completas del mismo documento en cajones distintos: si un cajon se quema, la otra copia sigue intacta. Ningun nivel de RAID reemplaza tener una copia adicional guardada en un lugar completamente distinto (backup real).

## Ejemplo base

```bash
# Crear un RAID 1 (mirror) con mdadm entre dos discos
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

cat /proc/mdstat              # estado actual del arreglo RAID, incluyendo sincronizacion

sudo mkfs.ext4 /dev/md0        # formatea el arreglo RAID como un unico dispositivo logico
sudo mdadm --detail /dev/md0    # informacion detallada del arreglo
```

## Como se usa en un proyecto real

1. Se elige el nivel de RAID segun la prioridad real del caso: RAID 1 o 10 cuando la disponibilidad ante fallo de disco es critica, RAID 5 cuando se busca balance entre redundancia y espacio util con presupuesto de discos limitado.
2. Se implementa siempre backup real y separado, sin importar que nivel de RAID se use, dado que RAID no protege contra errores humanos ni corrupcion logica de datos.
3. Se monitorea activamente el estado de arreglos RAID (`cat /proc/mdstat`, alertas ante degradacion) para detectar un disco fallado antes de que un segundo fallo cause perdida total de datos.
4. Se documenta el nivel de RAID usado y su justificacion para cada servidor, como parte de la documentacion de infraestructura de almacenamiento.
5. Se evita usar RAID 0 para cualquier dato que importe conservar, reservandolo unicamente para casos donde la perdida total es aceptable a cambio de performance (por ejemplo, cache temporal descartable).

## Criterio junior

Un junior debe entender la diferencia basica entre los niveles de RAID mas comunes y por que RAID 0 no ofrece proteccion contra fallo de disco.

## Criterio mid

Un mid debe configurar un arreglo RAID basico con `mdadm`, monitorear su estado, y explicar por que RAID no reemplaza al backup.

## Criterio senior

Un senior debe diseñar la estrategia de redundancia de almacenamiento de servidores criticos, eligiendo el nivel de RAID apropiado segun el balance de costo/performance/redundancia, y garantizar backup real independiente en toda la infraestructura.

## Errores comunes

- Confundir RAID con backup, asumiendo que un arreglo RAID protege contra borrado accidental o corrupcion de datos a nivel de aplicacion.
- Usar RAID 0 para datos importantes, sin entender que en realidad aumenta el riesgo de perdida total frente a un disco individual.
- No monitorear activamente el estado de un arreglo RAID, descubriendo un disco fallado solo cuando un segundo fallo ya causo perdida de datos.
- No documentar la configuracion de RAID de un servidor, dificultando el mantenimiento y reemplazo de discos fallados.

## Practica

- [ ] Investiga o simula la creacion de un arreglo RAID 1 con `mdadm`.
- [ ] Revisa el estado de un arreglo RAID con `cat /proc/mdstat`.
- [ ] Explica con tus palabras por que RAID no reemplaza al backup, con un ejemplo concreto de riesgo que RAID no cubre.
- [ ] Compara el espacio util disponible entre RAID 1 y RAID 5 para la misma cantidad de discos.
- [ ] Documenta el nivel de RAID que elegirias para un servidor de base de datos critico y por que.

## Preguntas de repaso

- Por que RAID 0 en realidad aumenta el riesgo de perdida de datos en vez de reducirlo?
- Que diferencia de eficiencia de espacio hay entre RAID 1 y RAID 5?
- Por que RAID no es un sustituto de una estrategia de backup real?
- Que monitorearias activamente en un arreglo RAID para prevenir perdida de datos ante fallos sucesivos?
