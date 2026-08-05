---
tags:
  - Linux
  - Bash
  - Linux-Shell
Creado: 2026-07-28
Relacionado:
  - Linea de Comandos y Shell
  - LINUX
---

# 03 - Redirecciones Pipes y Tuberias

## Que vas a aprender

En este capitulo vas a dominar el concepto que hace a Linux tan poderoso para automatizacion: **redirecciones** (`>`, `>>`, `<`) y **pipes** (`|`), que permiten combinar comandos simples en flujos de procesamiento complejos.

## Explicacion

Todo proceso en Linux tiene tres flujos estandar: **stdin** (entrada), **stdout** (salida normal), y **stderr** (salida de error), identificados por los descriptores de archivo 0, 1 y 2. Las **redirecciones** cambian a donde va cada flujo: `>` redirige stdout a un archivo (sobreescribiendo), `>>` lo agrega al final sin sobreescribir, `2>` redirige especificamente stderr, y `<` toma la entrada de un archivo en vez del teclado.

Un **pipe** (`|`) conecta el stdout de un comando directamente al stdin del siguiente, permitiendo encadenar comandos simples (cada uno haciendo una sola cosa bien) en una tuberia de procesamiento, sin necesitar archivos intermedios. Esta filosofia de "programas pequeños que hacen una cosa bien y se combinan" es central a la forma de trabajar en Unix/Linux.

## Modelo mental

Pensa stdin/stdout/stderr como tres canerias distintas de agua saliendo o entrando de una maquina: por defecto, la entrada viene del teclado y la salida (tanto normal como de error) va a la pantalla, pero podes redirigir cualquiera de esas canerias hacia otro lado (un archivo, o directamente a la entrada de otra maquina) sin que la maquina original necesite saberlo.

## Ejemplo base

```bash
echo "hola" > archivo.txt        # sobreescribe archivo.txt con "hola"
echo "mundo" >> archivo.txt       # agrega "mundo" al final, sin borrar lo anterior
comando_que_falla 2> errores.log  # solo los errores van al archivo, la salida normal se ve en pantalla
comando > salida.log 2>&1         # ambos, stdout y stderr, van al mismo archivo

cat archivo.txt | grep "mundo" | wc -l
# tuberia: cat imprime el archivo, grep filtra lineas con "mundo", wc -l cuenta esas lineas
```

## Como se usa en un proyecto real

1. Se separan logs de salida normal y de errores en archivos distintos (`> out.log 2> err.log`) cuando se necesita distinguir claramente exitos de fallos en un script automatizado.
2. Se combinan comandos simples con pipes para construir analisis de datos ad-hoc directamente en la terminal, sin necesitar escribir un script completo para tareas puntuales.
3. Se usa `2>&1` cuando se quiere capturar tanto salida normal como errores en un unico archivo, comun en logs de scripts de cron.
4. Se evita sobreescribir accidentalmente un archivo importante con `>` cuando en realidad se queria `>>` (agregar), verificando siempre la intencion antes de ejecutar.
5. Se construyen pipelines de procesamiento de texto combinando `grep`, `sed`, `awk` (ver capitulo de Editores y Manipulacion de Texto) encadenados con pipes.

## Criterio junior

Un junior debe poder redirigir la salida de un comando a un archivo, y encadenar dos comandos simples con un pipe.

## Criterio mid

Un mid debe distinguir cuando redirigir stdout, stderr, o ambos, y construir pipelines de varios comandos encadenados para tareas de procesamiento de texto reales.

## Criterio senior

Un senior debe diseñar scripts de automatizacion que manejen correctamente la separacion de salida normal y errores, facilitando el diagnostico automatizado de fallos en pipelines de CI/CD o tareas programadas.

## Errores comunes

- Usar `>` cuando en realidad se queria `>>`, sobreescribiendo accidentalmente contenido importante de un archivo.
- No redirigir stderr por separado en scripts de automatizacion, perdiendo la distincion entre exito y fallo real en los logs.
- Construir pipelines demasiado largos y dificiles de leer sin dividirlos en pasos intermedios cuando la complejidad lo justifica.
- No entender que un pipe conecta stdout con stdin, intentando encadenar comandos que no producen salida por stdout de forma util.

## Practica

- [ ] Redirige la salida de un comando a un archivo con `>`, y luego agrega mas contenido con `>>`.
- [ ] Redirige stdout y stderr a archivos separados para un comando que genere ambos tipos de salida.
- [ ] Construye un pipeline de al menos tres comandos encadenados con pipes para procesar texto.
- [ ] Usa `2>&1` para combinar ambos flujos en un unico archivo de log.
- [ ] Anota un caso real donde separar stdout y stderr te ayudaria a diagnosticar un problema.

## Preguntas de repaso

- Que representan los descriptores de archivo 0, 1 y 2?
- Que diferencia hay entre `>` y `>>`?
- Como conecta un pipe (`|`) dos comandos entre si?
- Por que separar stderr de stdout es util en scripts de automatizacion?
