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

# 01 - Permisos Unix rwx

## Que vas a aprender

En este capitulo vas a entender el modelo fundamental de permisos de Linux: **lectura (r), escritura (w), ejecucion (x)**, aplicados a tres niveles (dueño, grupo, otros), y como leer e interpretar la notacion que ves con `ls -l`.

## Explicacion

Cada archivo o directorio en Linux tiene permisos definidos para tres categorias de usuarios: el **dueño** (owner), el **grupo** asociado, y **otros** (cualquier otro usuario del sistema). Para cada categoria, se definen tres permisos: **r** (leer contenido de un archivo, o listar un directorio), **w** (modificar un archivo, o crear/borrar archivos dentro de un directorio), y **x** (ejecutar un archivo como programa, o "entrar" a un directorio con `cd`).

La salida de `ls -l` muestra esto como una cadena de 10 caracteres: el primero indica el tipo (`-` archivo, `d` directorio, `l` enlace simbolico), y los siguientes 9 se dividen en tres grupos de tres (`rwx` para dueño, grupo, otros), donde un `-` significa que ese permiso especifico esta denegado.

## Modelo mental

Pensa los permisos como tres llaves distintas para la misma puerta, entregadas a tres tipos de personas: una llave para el dueño de la casa, una llave compartida para la familia (grupo), y una llave mas limitada para visitas (otros). Cada llave puede o no abrir ciertas puertas (leer), permitir mover muebles (escribir), o permitir usar los electrodomesticos (ejecutar).

## Ejemplo base

```bash
ls -l archivo.sh
# -rwxr-xr-- 1 juber developers 1024 Jul 28 10:00 archivo.sh
#
# - : archivo regular
# rwx : el dueño (juber) puede leer, escribir y ejecutar
# r-x : el grupo (developers) puede leer y ejecutar, pero no escribir
# r-- : otros solo pueden leer
```

```bash
ls -ld /var/log        # permisos de un directorio (nota el flag -d para no listar su contenido)
```

## Como se usa en un proyecto real

1. Se verifica siempre los permisos con `ls -l` antes de asumir por que un usuario no puede acceder o modificar un archivo.
2. Se distingue el significado de `x` en un archivo (puede ejecutarse como programa) del significado de `x` en un directorio (puede "entrarse" con `cd`), que son conceptualmente distintos aunque compartan la misma letra.
3. Se evita dar permisos de escritura a "otros" en archivos sensibles, aplicando el principio de menor privilegio ya visto en el libro de Docker.
4. Se diagnostica un error "Permission denied" revisando primero los permisos con `ls -l` antes de buscar causas mas complejas.
5. Se documenta el modelo mental de rwx como base indispensable antes de aprender `chmod`/`chown` (siguiente tema).

## Criterio junior

Un junior debe poder leer e interpretar correctamente la salida de `ls -l`, identificando permisos de dueño, grupo y otros para un archivo dado.

## Criterio mid

Un mid debe diagnosticar errores de "Permission denied" revisando los permisos correctos segun el usuario que intenta la operacion, y entender la diferencia de `x` entre archivos y directorios.

## Criterio senior

Un senior debe diseñar esquemas de permisos para directorios compartidos entre multiples usuarios o servicios, aplicando el modelo rwx de forma consistente con el principio de menor privilegio en toda la infraestructura.

## Errores comunes

- Confundir el significado de `x` en un directorio (permite "entrar") con su significado en un archivo (permite ejecutarlo como programa).
- No revisar permisos con `ls -l` antes de asumir causas mas complejas ante un error de acceso denegado.
- Dar permisos de escritura innecesarios a "otros" en archivos que no deberian ser modificables por cualquier usuario del sistema.
- No entender que los permisos de un directorio afectan la capacidad de crear/borrar archivos dentro de el, independientemente de los permisos del archivo individual.

## Practica

- [ ] Lista varios archivos y directorios con `ls -l` e interpreta manualmente cada permiso mostrado.
- [ ] Identifica un archivo donde el grupo tiene permisos distintos al dueño y explica el efecto practico.
- [ ] Provoca un error de "Permission denied" a proposito e identifica la causa revisando permisos.
- [ ] Compara el efecto de `x` en un archivo ejecutable vs en un directorio.
- [ ] Anota un caso real donde ajustarias permisos siguiendo el principio de menor privilegio.

## Preguntas de repaso

- Que representan las tres categorias de permisos (dueño, grupo, otros)?
- Que diferencia practica tiene el permiso `x` aplicado a un archivo versus aplicado a un directorio?
- Como interpretarias la cadena `-rwxr-xr--` mostrada por `ls -l`?
- Por que es mala practica dar permisos de escritura a "otros" en archivos sensibles?
