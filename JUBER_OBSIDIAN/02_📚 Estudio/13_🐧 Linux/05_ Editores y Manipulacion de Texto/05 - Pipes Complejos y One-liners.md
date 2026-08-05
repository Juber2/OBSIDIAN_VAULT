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

# 05 - Pipes Complejos y One-liners

## Que vas a aprender

En este capitulo vas a integrar todo el capitulo construyendo **pipelines complejos**: combinaciones de `grep`, `sed`, `awk`, `sort`, `uniq`, y `cut` encadenadas para resolver tareas reales de analisis de logs y datos en una sola linea.

## Explicacion

El poder real de las herramientas de texto de Unix aparece al **combinarlas**: cada una hace una cosa especifica bien (filtrar, transformar, extraer columnas, ordenar, contar), y encadenandolas con pipes se construyen analisis sofisticados sin escribir un script completo. Herramientas complementarias no vistas aun en este capitulo: `sort` ordena lineas (`-n` numerico, `-r` reverso, `-k` por columna especifica), `uniq` elimina lineas duplicadas consecutivas (por eso casi siempre se usa despues de `sort`, que agrupa duplicados juntos), `uniq -c` cuenta ocurrencias de cada linea unica, y `cut` extrae columnas de forma mas simple que `awk` cuando el caso es trivial.

Un patron extremadamente comun es "top N de algo": extraer un campo relevante, ordenarlo, contar ocurrencias, y quedarse con los mas frecuentes, todo en una sola linea combinando estas herramientas.

## Modelo mental

Pensa un pipeline complejo como una linea de ensamblaje de una fabrica de analisis de datos: cada estacion (comando) recibe el material semi-procesado de la estacion anterior, le hace su transformacion especifica, y lo pasa a la siguiente, hasta que al final de la linea sale exactamente el reporte que necesitabas, sin que ninguna estacion individual necesite saber hacer todo el trabajo.

## Ejemplo base

```bash
# Top 5 IPs que mas requests hicieron en un log de acceso web
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -5

# Codigos de estado HTTP mas frecuentes en un log
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# Usuarios unicos que iniciaron sesion, ordenados alfabeticamente
grep "login successful" auth.log | awk '{print $5}' | sort -u

# Procesos unicos ejecutandose, extrayendo solo el nombre del comando
ps aux | awk '{print $11}' | sort | uniq -c | sort -rn | head -10
```

## Como se usa en un proyecto real

1. Se construyen one-liners de diagnostico ad-hoc directamente en la terminal para investigaciones puntuales, sin necesitar escribir un script completo cada vez.
2. Se usa el patron `sort | uniq -c | sort -rn` como formula estandar de "top N mas frecuente" aplicada a cualquier tipo de dato extraido de un log.
3. Se documentan los one-liners mas utiles y repetidos del equipo (por ejemplo, en un archivo de "comandos utiles" del proyecto) para no reinventarlos cada vez.
4. Se convierten pipelines que se repiten frecuentemente en scripts propios (ver capitulo de Scripting con Bash) cuando ya no son tareas puntuales sino recurrentes.
5. Se combinan estas herramientas con `grep`/`sed`/`awk` ya vistos, demostrando la filosofia Unix de programas pequeños que se combinan para resolver problemas complejos.

## Criterio junior

Un junior debe poder combinar dos o tres comandos simples con pipes para resolver una tarea basica de filtrado o conteo.

## Criterio mid

Un mid debe construir el patron "top N mas frecuente" (`sort | uniq -c | sort -rn`) aplicado a distintos tipos de datos extraidos de logs.

## Criterio senior

Un senior debe reconocer cuando un one-liner repetido deberia convertirse en un script mantenible, y documentar los pipelines de diagnostico mas utiles como herramientas compartidas del equipo.

## Errores comunes

- Usar `uniq` sin haber ordenado antes con `sort`, dado que `uniq` solo elimina duplicados **consecutivos**, no duplicados dispersos en el archivo.
- Construir un pipeline tan largo y complejo que se vuelve dificil de leer y mantener, sin dividirlo en pasos documentados cuando la complejidad lo justifica.
- No documentar one-liners utiles que se repiten frecuentemente, reinventando la rueda cada vez que se necesita el mismo analisis.
- Repetir manualmente el mismo pipeline complejo en vez de convertirlo en un script reutilizable cuando ya se volvio una tarea recurrente.

## Practica

- [ ] Construye el patron "top N mas frecuente" sobre un log de acceso web de ejemplo.
- [ ] Extrae usuarios unicos de un log de autenticacion, ordenados alfabeticamente.
- [ ] Combina al menos cuatro comandos distintos en un unico pipeline para una tarea real.
- [ ] Identifica un one-liner que uses frecuentemente y documentalo para tu equipo.
- [ ] Evalua si algun pipeline complejo que construiste deberia convertirse en un script mantenible.

## Preguntas de repaso

- Por que `uniq` casi siempre se usa despues de `sort` en un pipeline?
- Como construirias el patron de "top N mas frecuente" de un dato extraido de un log?
- Cuando un one-liner deberia convertirse en un script propio en vez de repetirse manualmente cada vez?
- Que filosofia de diseño de Unix demuestra la combinacion de herramientas pequeñas mediante pipes?
