---
tags:
  - Linux
  - Permisos
  - Linux-Permisos
Creado: 2026-07-28
Relacionado:
  - Gestion de Archivos y Permisos
  - LINUX
---

# 05 - Busqueda de Archivos find locate grep

## Que vas a aprender

En este capitulo vas a dominar las herramientas de busqueda mas importantes de Linux: **find** (busqueda de archivos por criterios), **locate** (busqueda rapida usando un indice), y **grep** aplicado a busqueda de contenido dentro de archivos.

## Explicacion

`find` recorre el filesystem en tiempo real segun criterios flexibles: nombre, tipo, tamaño, fecha de modificacion, permisos, y puede combinarse con `-exec` para ejecutar una accion sobre cada resultado encontrado. Es lento en filesystems grandes porque escanea en vivo, pero siempre refleja el estado actual exacto del sistema.

`locate` busca en una **base de datos precalculada** (actualizada periodicamente, tipicamente por un cron diario via `updatedb`), siendo mucho mas rapido que `find` pero con el riesgo de no reflejar archivos creados muy recientemente si el indice no se actualizo todavia. `grep -r` busca contenido de texto dentro de archivos de forma recursiva, distinto de `find` que busca por metadata del archivo (nombre, tamaño), no por su contenido interno.

## Modelo mental

Pensa `find` como recorrer fisicamente cada pasillo de una biblioteca enorme buscando un libro especifico (lento pero siempre exacto y actualizado), `locate` como consultar el catalogo indexado de la biblioteca (rapido, pero puede no tener registrado un libro que llego hace cinco minutos), y `grep -r` como no buscar el libro por su titulo, sino leer el contenido de cada libro buscando una frase especifica dentro.

## Ejemplo base

```bash
find /var/log -name "*.log" -mtime -7        # logs modificados en los ultimos 7 dias
find / -type f -size +100M 2>/dev/null        # archivos mayores a 100MB, ignorando errores de permiso
find . -name "*.tmp" -exec rm {} \;            # busca y elimina cada resultado

locate nginx.conf                              # busqueda rapida via indice precalculado
sudo updatedb                                  # actualiza manualmente el indice de locate

grep -r "TODO" ./src                           # busca la palabra "TODO" en todo el codigo, recursivo
grep -rn "error" /var/log/app.log              # con numero de linea (-n)
```

## Como se usa en un proyecto real

1. Se usa `find` para tareas de mantenimiento que necesitan exactitud en tiempo real, como limpiar archivos temporales viejos combinado con `-exec rm`.
2. Se usa `locate` para busquedas rapidas exploratorias donde no importa si un archivo creado hace segundos aun no aparece.
3. Se combina `grep -r` con herramientas de desarrollo para buscar patrones de codigo, TODOs, o mensajes de error especificos dentro de archivos de log o codigo fuente.
4. Se redirige stderr a `/dev/null` en busquedas de `find` sobre todo el sistema, para evitar que errores de "permission denied" en directorios inaccesibles ensucien la salida util.
5. Se combina `find -exec` con precaucion en operaciones destructivas, verificando primero con un `-print` o `echo` que criterio de busqueda coincide exactamente con lo esperado antes de ejecutar `rm`.

## Criterio junior

Un junior debe poder usar `find` con un criterio simple (nombre o tipo de archivo) y `grep -r` para buscar texto dentro de archivos.

## Criterio mid

Un mid debe combinar criterios avanzados de `find` (tamaño, fecha, permisos) con `-exec` para tareas de mantenimiento, y elegir entre `find` y `locate` segun la necesidad de exactitud vs velocidad.

## Criterio senior

Un senior debe automatizar tareas de mantenimiento del sistema basadas en `find` (limpieza de archivos viejos, auditoria de permisos) como parte de scripts de administracion regulares, verificando siempre su comportamiento antes de aplicarlos en produccion.

## Errores comunes

- Usar `find -exec rm` sin verificar antes que criterio de busqueda coincide exactamente con lo esperado, arriesgando eliminar archivos incorrectos.
- Confiar en `locate` para una busqueda que necesita reflejar archivos creados muy recientemente, sin saber que el indice puede estar desactualizado.
- No redirigir stderr a `/dev/null` en busquedas amplias de `find`, generando salida ilegible por errores de permiso en directorios inaccesibles.
- Usar `grep` sin `-r` esperando que busque recursivamente, cuando por defecto solo busca en el archivo o directorio especificado directamente.

## Practica

- [ ] Busca archivos modificados en los ultimos 7 dias en un directorio de logs con `find`.
- [ ] Busca archivos mayores a un tamaño especifico en todo el sistema, redirigiendo errores de permiso.
- [ ] Compara la velocidad de `find` vs `locate` para la misma busqueda en un sistema con muchos archivos.
- [ ] Busca una palabra especifica de forma recursiva en un proyecto de codigo con `grep -r`.
- [ ] Diseña (sin ejecutar destructivamente) un comando `find -exec` para limpiar archivos temporales viejos.

## Preguntas de repaso

- Por que `find` es mas lento pero mas exacto que `locate`?
- Que verificarias antes de ejecutar un `find -exec rm` sobre resultados de una busqueda?
- Que diferencia hay entre buscar por metadata de archivo (`find`) y buscar por contenido (`grep -r`)?
- Como evitarias que errores de permiso ensucien la salida de una busqueda amplia con `find`?
