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

# 02 - Condicionales y Loops

## Que vas a aprender

En este capitulo vas a aprender a controlar el flujo de un script Bash con **condicionales** (`if`/`elif`/`else`) y **loops** (`for`, `while`), incluyendo las pruebas de condicion mas comunes sobre archivos y strings.

## Explicacion

`if [ condicion ]; then ... fi` evalua una condicion y ejecuta un bloque solo si es verdadera; `[ ]` (o `test`) soporta comparaciones de strings (`=`, `!=`), numeros (`-eq`, `-gt`, `-lt`), y pruebas sobre archivos (`-f` si existe y es archivo regular, `-d` si existe y es directorio, `-x` si es ejecutable). La forma moderna `[[ ]]` es mas flexible y menos propensa a errores de sintaxis que `[ ]`, especialmente al comparar strings con espacios.

Los loops `for` iteran sobre una lista de elementos (archivos, palabras, un rango de numeros), mientras que `while` repite mientras una condicion siga siendo verdadera, util para leer un archivo linea por linea o esperar hasta que algo este listo (un patron ya visto conceptualmente en los health checks del libro de Docker).

## Modelo mental

Pensa un condicional como una bifurcacion en un camino: el script llega a un punto de decision y toma un camino distinto segun si la condicion es verdadera o falsa. Un loop `for` es como recorrer una lista de tareas pendientes una por una hasta terminarlas todas; un loop `while` es como seguir trabajando mientras una alarma especifica no haya sonado todavia.

## Ejemplo base

```bash
#!/bin/bash

archivo="/etc/nginx/nginx.conf"
if [[ -f "$archivo" ]]; then
    echo "El archivo existe"
elif [[ -d "$archivo" ]]; then
    echo "Es un directorio, no un archivo"
else
    echo "No existe"
fi

for servicio in nginx postgresql redis; do
    echo "Verificando $servicio..."
    systemctl is-active --quiet "$servicio" && echo "OK" || echo "CAIDO"
done

contador=0
while [[ $contador -lt 5 ]]; do
    echo "Intento $contador"
    contador=$((contador + 1))
done
```

## Como se usa en un proyecto real

1. Se usa `[[ ]]` en vez de `[ ]` por defecto en scripts nuevos, dado que es mas robusto ante strings con espacios o vacios.
2. Se verifica la existencia de archivos y directorios criticos (`-f`, `-d`) antes de operar sobre ellos, evitando errores confusos mas adelante en el script.
3. Se usa `for` para iterar sobre listas conocidas (servicios a verificar, archivos a procesar), y `while` para esperar condiciones que tardan en cumplirse (similar a `depends_on: condition: service_healthy` visto en Docker).
4. Se combina `systemctl is-active --quiet` con condicionales para verificar el estado de servicios dentro de scripts de monitoreo o despliegue.
5. Se documenta la logica de control de flujo compleja con comentarios, especialmente cuando hay condiciones anidadas.

## Criterio junior

Un junior debe poder escribir un `if`/`else` simple y un loop `for` basico que itere sobre una lista de elementos.

## Criterio mid

Un mid debe usar pruebas de archivo (`-f`, `-d`, `-x`) para validaciones robustas, y combinar `while` con condiciones que dependen del resultado de comandos.

## Criterio senior

Un senior debe escribir scripts de administracion con control de flujo robusto (validaciones, manejo de casos borde) que sirvan como herramientas confiables de automatizacion para todo el equipo.

## Errores comunes

- Usar `[ ]` con variables que pueden estar vacias o contener espacios sin comillas, causando errores de sintaxis confusos.
- No verificar la existencia de un archivo antes de operar sobre el, generando errores poco claros mas adelante en el script.
- Confundir `=` (asignacion) con `-eq` (comparacion numerica) o viceversa, generando comparaciones incorrectas.
- Crear un loop `while` sin una condicion de salida clara, arriesgando un loop infinito si la condicion nunca se cumple.

## Practica

- [ ] Escribe un `if`/`elif`/`else` que verifique si un archivo existe, es un directorio, o no existe.
- [ ] Escribe un loop `for` que itere sobre una lista de servicios y verifique su estado con `systemctl`.
- [ ] Escribe un loop `while` con una condicion de salida clara.
- [ ] Compara `[ ]` y `[[ ]]` con una variable que contenga espacios.
- [ ] Anota una checklist de validaciones que aplicarias al inicio de cualquier script critico.

## Preguntas de repaso

- Por que `[[ ]]` es preferible a `[ ]` en scripts modernos de Bash?
- Que verifican las pruebas `-f`, `-d` y `-x` sobre un archivo?
- Cuando conviene usar `for` sobre `while`, y viceversa?
- Que riesgo tiene un loop `while` sin una condicion de salida clara?
