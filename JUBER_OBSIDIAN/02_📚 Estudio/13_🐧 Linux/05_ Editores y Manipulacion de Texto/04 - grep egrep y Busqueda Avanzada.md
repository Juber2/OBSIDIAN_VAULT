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

# 04 - grep egrep y Busqueda Avanzada

## Que vas a aprender

En este capitulo vas a profundizar en `grep` (ya usado de forma basica en el capitulo de Gestion de Archivos): opciones avanzadas, la diferencia entre expresiones regulares basicas y **extendidas** (`egrep`/`grep -E`), y su rol como filtro en pipelines de diagnostico.

## Explicacion

`grep` basico soporta un subconjunto limitado de expresiones regulares (Basic Regular Expressions, BRE), donde caracteres como `+`, `?`, `|` necesitan escaparse con `\` para tener significado especial. `egrep` (equivalente a `grep -E`) usa Extended Regular Expressions (ERE), donde esos mismos caracteres funcionan directamente sin escapar, permitiendo patrones como alternancia (`opcion1|opcion2`) de forma mas legible.

Opciones clave del dia a dia: `-i` (ignorar mayusculas/minusculas), `-v` (invertir, mostrar lineas que NO coinciden), `-c` (contar coincidencias en vez de mostrarlas), `-n` (mostrar numero de linea), `-A`/`-B`/`-C` (mostrar N lineas de contexto despues/antes/ambos de cada coincidencia, extremadamente util al revisar logs alrededor de un error especifico).

## Modelo mental

Pensa `grep` como un colador que solo deja pasar las lineas que coinciden con lo que buscas, descartando todo el resto. Las opciones de contexto (`-A`/`-B`/`-C`) son como pedirle al colador que ademas de la linea que coincide, te muestre unas cuantas lineas antes y despues, dandote el contexto completo alrededor del hallazgo, no solo la linea aislada.

## Ejemplo base

```bash
grep -i "error" app.log             # busca "error" sin importar mayusculas/minusculas
grep -v "^#" config.txt              # muestra todo EXCEPTO lineas de comentario
grep -c "warning" app.log            # cuenta cuantas veces aparece "warning"
grep -n "TODO" *.js                  # con numero de linea, en varios archivos

grep -A 3 -B 1 "Exception" app.log   # 1 linea antes y 3 despues de cada "Exception"

grep -E "error|warning|critical" app.log   # ERE: alternancia sin escapar el pipe
egrep "error|warning|critical" app.log     # equivalente a grep -E
```

## Como se usa en un proyecto real

1. Se usa `grep -i` habitualmente al buscar en logs, dado que niveles de log a veces varian entre mayusculas y minusculas segun la herramienta que los genero.
2. Se usa `-A`/`-B`/`-C` como primera herramienta al investigar un error en logs, dado que el contexto alrededor de una excepcion suele ser tan importante como la linea del error en si.
3. Se combina `grep -E` con alternancia para buscar multiples patrones relacionados en una sola pasada, en vez de correr `grep` varias veces por separado.
4. Se usa `grep -v` para filtrar ruido conocido de un log (por ejemplo, excluir health checks exitosos) y quedarse solo con lo relevante.
5. Se integra `grep` en pipelines con `awk`/`sed` (capitulos anteriores), cada herramienta contribuyendo la parte del procesamiento que mejor resuelve.

## Criterio junior

Un junior debe poder usar `grep` con opciones basicas (`-i`, `-v`, `-n`) para buscar texto en archivos y logs.

## Criterio mid

Un mid debe usar `-A`/`-B`/`-C` para obtener contexto alrededor de coincidencias, y `grep -E` para patrones con alternancia.

## Criterio senior

Un senior debe construir pipelines de diagnostico eficientes combinando `grep` con otras herramientas de texto, como parte del flujo estandar de investigacion de incidentes en logs de produccion.

## Errores comunes

- No usar `-i` al buscar en logs con capitalizacion inconsistente, perdiendo coincidencias relevantes por diferencia de mayusculas/minusculas.
- Ejecutar `grep` multiples veces por separado en vez de usar `-E` con alternancia para buscar varios patrones relacionados de una vez.
- No aprovechar `-A`/`-B`/`-C` al investigar un error, perdiendo el contexto que explicaria la causa del problema.
- Confundir expresiones regulares basicas y extendidas, escribiendo un patron que no funciona como se espera por no escapar (o escapar de mas) caracteres especiales.

## Practica

- [ ] Busca un texto en un archivo de log ignorando mayusculas/minusculas con `-i`.
- [ ] Usa `-v` para filtrar y excluir lineas que coincidan con un patron conocido de ruido.
- [ ] Investiga un error simulado usando `-A`/`-B`/`-C` para ver contexto alrededor.
- [ ] Busca multiples patrones relacionados en una sola pasada con `grep -E` y alternancia.
- [ ] Construye un pipeline combinando `grep` con `awk` o `sed` para una tarea real de tu trabajo.

## Preguntas de repaso

- Que diferencia hay entre expresiones regulares basicas (BRE) y extendidas (ERE)?
- Que hace la opcion `-v` de grep, y cuando es util?
- Como usarias `-A`/`-B`/`-C` para investigar un error dentro de un log grande?
- Por que `grep -E` con alternancia es mas eficiente que correr `grep` varias veces por separado?
