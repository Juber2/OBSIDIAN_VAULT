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

# 04 - Logs y Debugging del Kernel dmesg

## Que vas a aprender

En este capitulo vas a aprender a diagnosticar problemas a nivel de **kernel** con **`dmesg`**, incluyendo la deteccion del temido **OOM Killer** (ya visto conceptualmente en el capitulo de Costos del libro de Docker), y problemas de hardware.

## Explicacion

`dmesg` muestra el **buffer de mensajes del kernel**, donde se registran eventos de bajo nivel que nunca llegan a los logs de aplicacion normales: deteccion de hardware al arrancar, errores de disco, problemas de red a nivel de driver, y **eventos del OOM Killer** (Out-Of-Memory Killer), el mecanismo del kernel que mata forzosamente un proceso cuando el sistema se queda completamente sin memoria disponible (incluyendo swap), eligiendo tipicamente el proceso que mas memoria esta usando en ese momento.

Cuando un servicio "simplemente desaparece" sin ningun error visible en sus propios logs de aplicacion, revisar `dmesg | grep -i "killed process"` es el primer paso para confirmar si el OOM Killer fue el responsable, informacion que la aplicacion misma nunca podria loggear (fue terminada abruptamente, sin oportunidad de escribir nada). Esto conecta directamente con el dimensionamiento de limites de memoria visto en el capitulo Senior del libro de Docker: el mismo mecanismo del kernel aplica tanto a procesos normales del host como a procesos dentro de containers que exceden su limite de memoria asignado.

## Modelo mental

Pensa `dmesg` como la caja negra de un avion: registra eventos criticos de bajo nivel que las herramientas normales de cabina (logs de aplicacion) nunca capturan, siendo la unica fuente de verdad cuando algo fallo de forma tan abrupta que el sistema normal de registro ni siquiera tuvo oportunidad de anotar que paso.

## Ejemplo base

```bash
dmesg | tail -50                       # ultimos 50 mensajes del kernel
dmesg -T | grep -i "killed process"     # busca eventos del OOM Killer, con timestamps legibles
dmesg -T | grep -i error                # busca cualquier mensaje de error del kernel
dmesg --level=err,warn                  # filtra solo por nivel de severidad

journalctl -k -b                         # equivalente moderno via journalctl, del boot actual
```

## Como se usa en un proyecto real

1. Se revisa `dmesg | grep -i "killed process"` como primer paso cuando un servicio desaparece sin explicacion en sus propios logs de aplicacion.
2. Se conecta un evento de OOM Killer confirmado con el dimensionamiento de recursos del sistema o de containers especificos (ver capitulo de Costos y Performance del libro de Docker), ajustando limites segun corresponda.
3. Se revisan mensajes de error de disco o hardware en `dmesg` cuando se sospecha un problema fisico subyacente detras de sintomas de performance degradada.
4. Se combina `dmesg` con `journalctl -k` (el equivalente estructurado via el journal de systemd) segun cual sea mas comodo de filtrar para el caso especifico.
5. Se documenta cualquier hallazgo de OOM Killer en produccion como incidente a investigar, dado que representa un sistema operando en un limite critico de recursos.

## Criterio junior

Un junior debe poder revisar `dmesg` para buscar mensajes recientes del kernel, y saber que el OOM Killer puede terminar procesos sin que ellos mismos lo registren.

## Criterio mid

Un mid debe diagnosticar un servicio desaparecido inesperadamente revisando `dmesg` por eventos de OOM Killer, y correlacionarlo con el consumo de memoria del sistema.

## Criterio senior

Un senior debe usar hallazgos de OOM Killer en `dmesg` para justificar ajustes de dimensionamiento de recursos (memoria fisica del servidor, limites de containers), previniendo recurrencia del problema en vez de solo reaccionar cada vez que ocurre.

## Errores comunes

- No revisar `dmesg` cuando un servicio desaparece sin explicacion, perdiendo la unica fuente de informacion sobre por que realmente termino (OOM Killer).
- Confundir un proceso terminado por el OOM Killer con un crash de aplicacion normal, investigando en el lugar equivocado.
- No conectar un evento de OOM Killer con una accion de dimensionamiento correctiva, dejando que el mismo problema se repita.
- No revisar mensajes de hardware/disco en `dmesg` cuando se sospecha un problema fisico subyacente detras de sintomas de performance.

## Practica

- [ ] Revisa `dmesg` en tu sistema e identifica que tipo de eventos registra.
- [ ] Busca especificamente eventos de OOM Killer con `grep -i "killed process"`.
- [ ] Compara `dmesg` con `journalctl -k` para el mismo tipo de informacion.
- [ ] Simula (con precaucion, en un entorno de practica) una situacion de memoria agotada y observa si el OOM Killer actua.
- [ ] Documenta como conectarias un hallazgo de OOM Killer con una accion de dimensionamiento de recursos.

## Preguntas de repaso

- Que informacion registra `dmesg` que no aparece en logs de aplicacion normales?
- Que es el OOM Killer y por que un proceso terminado por el nunca puede registrar su propia terminacion?
- Como conectarias un evento de OOM Killer en `dmesg` con una decision de dimensionamiento de recursos?
- Que equivalente moderno de `dmesg` ofrece journalctl?
