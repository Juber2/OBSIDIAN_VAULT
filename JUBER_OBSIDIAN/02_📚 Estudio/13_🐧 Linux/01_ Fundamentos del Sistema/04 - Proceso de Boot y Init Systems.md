---
tags:
  - Linux
  - Sistemas
  - Linux-Fundamentos
Creado: 2026-07-28
Relacionado:
  - Fundamentos del Sistema
  - LINUX
---

# 04 - Proceso de Boot y Init Systems

## Que vas a aprender

En este capitulo vas a entender que ocurre desde que un servidor Linux se enciende hasta que esta listo para usarse: **BIOS/UEFI**, **bootloader** (GRUB), carga del **kernel**, y el rol de **systemd** como sistema de init moderno (ya usado indirectamente en el capitulo de systemd de gestion de procesos).

## Explicacion

El proceso de boot sigue una secuencia: el firmware (**BIOS** o **UEFI**, su sucesor moderno) inicializa el hardware basico y busca un bootloader; **GRUB** (el bootloader mas comun en Linux) carga el kernel en memoria y le pasa el control; el kernel se inicializa, monta el filesystem raiz, y finalmente ejecuta el primer proceso de espacio de usuario (PID 1), que en la mayoria de distribuciones modernas es **systemd**.

Systemd es responsable de arrancar todos los demas servicios del sistema en el orden correcto (respetando dependencias declaradas), reemplazando al sistema de init tradicional basado en scripts secuenciales (SysV init) que dominaba distribuciones mas antiguas. Entender esta cadena ayuda a diagnosticar problemas de arranque (un servidor que no bootea, un servicio que no inicia correctamente) identificando en que etapa especifica esta el problema.

## Modelo mental

Pensa el proceso de boot como la rutina matutina de una fabrica: primero se enciende la electricidad general (firmware), despues un supervisor de turno abre las puertas y llama al gerente de planta (bootloader carga el kernel), el gerente de planta organiza el edificio (kernel monta el filesystem), y finalmente un coordinador general (systemd) va llamando a cada departamento en el orden correcto segun que necesita que otro departamento este listo primero.

## Ejemplo base

```bash
systemd-analyze                    # tiempo total de boot, desglosado
systemd-analyze blame              # que servicio tardo mas en arrancar
systemctl list-units --type=service --state=failed   # servicios que fallaron al arrancar
journalctl -b                      # logs completos del boot actual
```

## Como se usa en un proyecto real

1. Se usa `systemd-analyze blame` para diagnosticar por que un servidor tarda mucho en arrancar, identificando el servicio especifico responsable.
2. Se revisan servicios fallidos despues de cada reinicio con `systemctl --failed`, como parte de una verificacion post-reinicio estandar.
3. Se investiga `journalctl -b` cuando un servidor no arranco correctamente, para ver exactamente en que punto de la secuencia de boot ocurrio el problema.
4. Se documenta el orden de dependencias entre servicios propios (definido en las unidades de systemd) para servicios que dependen entre si al arrancar.
5. Se entiende esta secuencia como base para diagnosticar problemas mas profundos que un simple "el servicio no responde".

## Criterio junior

Un junior debe poder nombrar las etapas basicas del proceso de boot (firmware, bootloader, kernel, init) y usar `journalctl -b` para ver logs del ultimo arranque.

## Criterio mid

Un mid debe usar `systemd-analyze` para diagnosticar tiempos de boot lentos y `systemctl --failed` para identificar servicios que no arrancaron correctamente.

## Criterio senior

Un senior debe diseñar dependencias correctas entre servicios propios en systemd, optimizar tiempos de boot en servidores criticos, y diagnosticar problemas complejos de arranque correlacionando logs de distintas etapas de la secuencia.

## Errores comunes

- No revisar servicios fallidos despues de un reinicio, asumiendo que todo arranco correctamente sin verificarlo.
- No usar `systemd-analyze blame` ante un boot lento, perdiendo tiempo adivinando en vez de medir la causa real.
- No declarar dependencias correctas entre servicios propios, causando fallos intermitentes cuando un servicio arranca antes de que su dependencia este lista.
- Confundir un problema de aplicacion con un problema de la secuencia de boot, perdiendo tiempo de diagnostico en el lugar equivocado.

## Practica

- [ ] Ejecuta `systemd-analyze` y `systemd-analyze blame` en un sistema real o de practica.
- [ ] Verifica si hay servicios fallidos con `systemctl list-units --type=service --state=failed`.
- [ ] Revisa los logs del ultimo boot con `journalctl -b`.
- [ ] Investiga como se declaran dependencias entre unidades de systemd (`After=`, `Requires=`).
- [ ] Anota como diagnosticarias un servidor que tarda anormalmente en arrancar.

## Preguntas de repaso

- Que hace el bootloader (GRUB) entre el firmware y el kernel?
- Por que systemd reemplazo al sistema de init tradicional basado en scripts secuenciales?
- Como usarias `systemd-analyze blame` para diagnosticar un boot lento?
- Que comando revisarias primero ante la sospecha de que un servicio no arranco correctamente?
