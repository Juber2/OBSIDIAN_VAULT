---
tags:
  - Linux
  - Bash
  - Linux-Texto
Creado: 2026-07-28
Relacionado:
  - Editores y Manipulacion de Texto
  - LINUX
---

# 03 - Awk para Procesamiento de Datos

## Que vas a aprender

En este capitulo vas a aprender **awk**, una herramienta orientada a procesar texto **estructurado en columnas** (como logs delimitados, CSVs simples, o salida de otros comandos), mucho mas poderosa que `sed` para este tipo de tareas.

## Explicacion

Mientras `sed` piensa en terminos de lineas completas y patrones de texto, `awk` piensa en terminos de **registros** (por defecto, cada linea) divididos en **campos** (por defecto, separados por espacios), accesibles como `$1`, `$2`, etc., con `$0` representando la linea completa. Esto lo hace ideal para extraer columnas especificas, sumar valores numericos de una columna, o filtrar filas segun una condicion sobre un campo especifico.

Awk tiene su propio mini-lenguaje de programacion: soporta variables, condicionales, loops, y bloques especiales `BEGIN`/`END` que se ejecutan antes/despues de procesar todas las lineas, utiles para inicializar contadores o imprimir totales acumulados al final del procesamiento.

## Modelo mental

Pensa `awk` como una hoja de calculo que procesa texto: cada linea de entrada es una fila, y automaticamente se divide en columnas ($1, $2, $3...) sin que vos tengas que parsearlas manualmente, permitiendote hacer operaciones (sumar, filtrar, reformatear) sobre columnas especificas como lo harias con formulas en Excel.

## Ejemplo base

```bash
# ps aux tiene columnas: USER PID %CPU %MEM ... COMMAND
ps aux | awk '{print $2, $11}'          # solo PID y comando
ps aux | awk '$3 > 5.0 {print $2, $3}'  # procesos con mas de 5% de CPU

echo "juber:developers:1000" | awk -F: '{print $1}'   # usa ":" como separador de campo

# Sumar una columna numerica de un archivo, con total al final
awk '{suma += $2} END {print "Total:", suma}' datos.txt
```

## Como se usa en un proyecto real

1. Se combina `awk` con comandos como `ps`, `df`, o logs delimitados para extraer y procesar solo la informacion relevante de una salida con muchas columnas.
2. Se usa `-F` para especificar un separador de campo distinto al espacio por defecto, comun al procesar archivos delimitados por comas o dos puntos (como `/etc/passwd`).
3. Se aprovechan los bloques `BEGIN`/`END` para calcular totales, promedios o contadores acumulados al procesar archivos de logs o metricas.
4. Se combina `awk` con `grep` y `sed` en pipelines (visto en capitulos anteriores), cada herramienta haciendo la parte que mejor resuelve.
5. Se usa `awk` en scripts de monitoreo simples para extraer y alertar sobre valores especificos de comandos del sistema (como uso de CPU de un proceso especifico).

## Criterio junior

Un junior debe poder extraer una columna especifica de una salida de comando usando `awk '{print $N}'`.

## Criterio mid

Un mid debe filtrar filas segun una condicion sobre un campo numerico, y usar `-F` para procesar archivos con separadores distintos al espacio.

## Criterio senior

Un senior debe escribir scripts `awk` mas complejos con bloques `BEGIN`/`END` para agregaciones (sumas, promedios, conteos) como parte de herramientas propias de monitoreo o analisis de logs.

## Errores comunes

- Usar `sed` para tareas que serian mucho mas simples con `awk` (extraer columnas, filtrar por valor numerico), eligiendo la herramienta menos apropiada para el caso.
- No especificar `-F` al procesar archivos con separadores distintos al espacio, obteniendo columnas mal divididas.
- No aprovechar `BEGIN`/`END` para calculos acumulados, procesando manualmente lo que `awk` podria hacer en un unico comando.
- Confundir `$0` (linea completa) con `$1` (primer campo), generando resultados inesperados.

## Practica

- [ ] Extrae una columna especifica de la salida de `ps aux` usando `awk`.
- [ ] Filtra procesos que superen un umbral de CPU usando una condicion en `awk`.
- [ ] Procesa un archivo delimitado por `:` usando `-F` para separar campos correctamente.
- [ ] Suma una columna numerica de un archivo usando el bloque `END`.
- [ ] Anota un caso real donde `awk` seria mas apropiado que `sed` para tu tarea.

## Preguntas de repaso

- Que diferencia fundamental hay entre `sed` (orientado a lineas) y `awk` (orientado a campos/columnas)?
- Como accederias al segundo campo de cada linea con `awk`?
- Para que sirven los bloques `BEGIN` y `END` en un script `awk`?
- Cuando elegirias `awk` sobre `sed` para una tarea especifica?
