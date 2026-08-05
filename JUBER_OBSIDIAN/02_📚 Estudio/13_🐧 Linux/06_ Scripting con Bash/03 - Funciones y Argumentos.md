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

# 03 - Funciones y Argumentos

## Que vas a aprender

En este capitulo vas a aprender a escribir **funciones** en Bash para organizar scripts complejos en piezas reutilizables, y a manejar **argumentos** pasados tanto a un script completo como a una funcion especifica.

## Explicacion

Una funcion en Bash (`nombre_funcion() { ... }`) agrupa comandos reutilizables bajo un nombre, evitando repetir la misma logica en varios lugares del script, el mismo principio de organizacion visto en cualquier lenguaje de programacion. Dentro de una funcion, los argumentos recibidos se acceden igual que los argumentos de un script completo: `$1`, `$2`, etc. para argumentos posicionales, `$#` para la cantidad total de argumentos, y `$@` para todos los argumentos como lista.

Un detalle importante para scripts robustos es declarar variables locales dentro de funciones con `local`, evitando que modifiquen accidentalmente variables globales del script con el mismo nombre. Las funciones pueden retornar un codigo de salida (`return`, solo numeros 0-255, por convencion 0 significa exito) o "retornar" datos via `echo` capturado con command substitution desde quien la llama.

## Modelo mental

Pensa una funcion como una herramienta especializada guardada en una caja de herramientas: en vez de explicar paso a paso como hacer una tarea cada vez que la necesitas, simplemente "usas la herramienta" (llamas a la funcion) pasandole lo que necesita (argumentos), y confias en que hace su trabajo correctamente sin tener que repensar los pasos internos cada vez.

## Ejemplo base

```bash
#!/bin/bash

log() {
    local nivel=$1
    local mensaje=$2
    echo "[$(date +%H:%M:%S)] [$nivel] $mensaje"
}

verificar_servicio() {
    local nombre=$1
    if systemctl is-active --quiet "$nombre"; then
        log "INFO" "$nombre esta activo"
        return 0
    else
        log "ERROR" "$nombre esta caido"
        return 1
    fi
}

log "INFO" "Iniciando script con $# argumentos: $@"
verificar_servicio "nginx"
echo "Codigo de salida: $?"     # $? captura el ultimo codigo de retorno
```

## Como se usa en un proyecto real

1. Se extraen bloques de logica repetida (como logging consistente, validaciones comunes) a funciones reutilizables al inicio del script.
2. Se declara `local` para toda variable dentro de una funcion, evitando efectos secundarios inesperados sobre variables globales del script.
3. Se usa `$#` para validar que un script recibio la cantidad correcta de argumentos antes de continuar, saliendo con un mensaje claro si falta alguno.
4. Se usan codigos de retorno (`return`) de forma consistente (0 para exito, distinto de 0 para error especifico) permitiendo que quien llama a la funcion reaccione correctamente.
5. Se organiza un script complejo con funciones bien nombradas, haciendo que el flujo principal del script (al final del archivo) sea legible como una lista de pasos de alto nivel.

## Criterio junior

Un junior debe poder escribir una funcion simple, llamarla con argumentos, y acceder a esos argumentos con `$1`, `$2` dentro de la funcion.

## Criterio mid

Un mid debe usar `local` correctamente dentro de funciones, y manejar codigos de retorno para que el script principal reaccione segun el resultado de una funcion.

## Criterio senior

Un senior debe estructurar scripts complejos completamente en funciones bien organizadas y nombradas, con validacion de argumentos consistente, sirviendo como plantilla de calidad para el equipo.

## Errores comunes

- No usar `local` dentro de funciones, causando que una variable modifique accidentalmente una variable global del mismo nombre en otra parte del script.
- No validar `$#` antes de usar argumentos posicionales, causando errores confusos cuando el script se llama sin los argumentos esperados.
- Confundir el proposito de `return` (codigo de salida numerico 0-255) con intentar "retornar" un string complejo directamente, cuando eso requiere `echo` + command substitution.
- Escribir scripts largos sin ninguna funcion, mezclando logica repetida disperse por todo el archivo sin organizacion.

## Practica

- [ ] Escribe una funcion de logging reutilizable que reciba nivel y mensaje como argumentos.
- [ ] Escribe una funcion que valide la existencia de un archivo y retorne un codigo de salida apropiado.
- [ ] Valida `$#` al inicio de un script para asegurar que recibio los argumentos esperados.
- [ ] Practica usar `local` dentro de una funcion y verifica que no afecta variables globales del mismo nombre.
- [ ] Reorganiza un script largo existente extrayendo logica repetida a funciones.

## Preguntas de repaso

- Que problema previene declarar variables con `local` dentro de una funcion?
- Como accederias al primer y segundo argumento recibido por una funcion?
- Que diferencia hay entre usar `return` para un codigo de salida y usar `echo` para "retornar" datos de una funcion?
- Por que validar `$#` al inicio de un script previene errores confusos mas adelante?
