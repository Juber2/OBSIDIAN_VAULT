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

# 04 - Manejo de Errores en Scripts

## Que vas a aprender

En este capitulo vas a aprender a escribir scripts Bash **robustos**: las opciones `set -e`/`set -u`/`set -o pipefail`, manejo de codigos de salida, y `trap` para limpieza garantizada ante errores o interrupciones.

## Explicacion

Por defecto, Bash **continua ejecutando** un script aunque un comando falle, lo cual puede causar que un script siga corriendo en un estado inconsistente sin que nadie lo note. `set -e` hace que el script se detenga inmediatamente ante cualquier comando que falle (retorne codigo distinto de 0). `set -u` trata el uso de una variable no definida como error, previniendo bugs silenciosos por typos en nombres de variables. `set -o pipefail` hace que un pipeline completo falle si **cualquier** comando en la cadena falla, no solo si falla el ultimo (comportamiento por defecto sin esta opcion).

`trap` (ya visto de forma introductoria en el capitulo de Señales) permite ejecutar codigo de limpieza garantizado cuando el script termina, sin importar si termino exitosamente, por error, o por una señal como `Ctrl+C` (`trap 'limpieza' EXIT`), asegurando que recursos temporales (archivos, locks) siempre se liberen correctamente.

## Modelo mental

Pensa `set -e` como un empleado que, al primer error serio en una tarea, para inmediatamente y avisa en vez de seguir trabajando como si nada hubiera pasado y potencialmente empeorar la situacion. `trap` es como asegurarte de que, sin importar como termine el turno de trabajo (bien, mal, o interrumpido a la fuerza), siempre se apaguen las luces y se cierre con llave antes de irse.

## Ejemplo base

```bash
#!/bin/bash
set -euo pipefail   # combinacion estandar de robustez

archivo_temporal=$(mktemp)
trap 'rm -f "$archivo_temporal"; echo "Limpieza completada"' EXIT

echo "Procesando..." > "$archivo_temporal"
comando_que_puede_fallar   # si falla, el script se detiene aca gracias a set -e
echo "Esto no se ejecuta si el comando anterior fallo"
```

```bash
# pipefail en accion
comando_que_falla | grep "algo"
echo $?   # sin pipefail, muestra el exit code de grep, NO del comando que realmente fallo
```

## Como se usa en un proyecto real

1. Se agrega `set -euo pipefail` al inicio de todo script de produccion o de despliegue, como practica estandar de robustez.
2. Se usa `trap ... EXIT` para garantizar limpieza de archivos temporales o locks, sin importar como termine el script.
3. Se maneja explicitamente el caso de comandos que "deberian poder fallar" sin detener el script (por ejemplo, verificar si algo existe) usando `|| true` o estructuras condicionales, dado que con `set -e` cualquier fallo detiene todo por defecto.
4. Se verifica el codigo de salida de operaciones criticas explicitamente (`if ! comando; then ...`) cuando se necesita manejar el error de forma mas especifica que simplemente detener el script.
5. Se prueban los scripts simulando fallos intencionales para verificar que el manejo de errores funciona como se espera antes de confiar en ellos en produccion.

## Criterio junior

Un junior debe poder agregar `set -euo pipefail` a un script y entender conceptualmente que hace cada opcion.

## Criterio mid

Un mid debe usar `trap` para garantizar limpieza de recursos temporales, y manejar explicitamente comandos que pueden fallar de forma esperada sin detener todo el script.

## Criterio senior

Un senior debe establecer `set -euo pipefail` y patrones de manejo de errores como estandar obligatorio en todos los scripts de infraestructura del equipo, revisando en code review que se respete consistentemente.

## Errores comunes

- No usar `set -e`, dejando que un script continue ejecutando pasos posteriores despues de que algo critico ya fallo, en un estado inconsistente.
- No usar `pipefail`, perdiendo visibilidad de fallos en pipelines donde solo el ultimo comando "exitoso" determina el codigo de salida reportado.
- Usar `set -e` sin manejar explicitamente comandos que pueden fallar de forma esperada, causando que el script se detenga donde en realidad no deberia.
- No usar `trap` para limpieza, dejando archivos temporales o locks huerfanos cuando un script termina de forma inesperada.

## Practica

- [ ] Agrega `set -euo pipefail` a un script existente y observa el cambio de comportamiento ante un fallo simulado.
- [ ] Usa `trap ... EXIT` para garantizar limpieza de un archivo temporal en un script de practica.
- [ ] Demuestra el problema que `pipefail` resuelve con un pipeline donde el comando intermedio falla.
- [ ] Maneja explicitamente un comando que puede fallar de forma esperada sin detener el resto del script.
- [ ] Simula distintos tipos de fallo en un script real para verificar que su manejo de errores es robusto.

## Preguntas de repaso

- Que hace `set -e` y que riesgo previene en un script?
- Que problema especifico resuelve `set -o pipefail` que no resuelve el comportamiento por defecto?
- Para que sirve `trap ... EXIT` y por que es mas confiable que poner la limpieza solo al final del script?
- Como manejarias un comando que puede fallar de forma esperada en un script con `set -e` activo?
