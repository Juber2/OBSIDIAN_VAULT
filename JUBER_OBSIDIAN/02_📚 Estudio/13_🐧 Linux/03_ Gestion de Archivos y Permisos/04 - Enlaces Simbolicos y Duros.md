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

# 04 - Enlaces Simbolicos y Duros

## Que vas a aprender

En este capitulo vas a entender la diferencia entre **enlaces simbolicos** (symlinks) y **enlaces duros** (hard links), dos formas distintas de que un archivo tenga mas de un "nombre" en el sistema de archivos.

## Explicacion

Un **enlace duro** (`ln origen destino`) crea una segunda entrada que apunta directamente al mismo **inodo** (la estructura interna del filesystem que representa el contenido real del archivo) que el archivo original: ambos nombres son completamente equivalentes, y el contenido solo se elimina realmente cuando **todos** los enlaces duros hacia el son borrados. Un enlace duro no puede cruzar sistemas de archivos distintos (particiones o discos diferentes) ni apuntar a un directorio.

Un **enlace simbolico** (`ln -s origen destino`, tambien llamado symlink) es un tipo especial de archivo que simplemente contiene la **ruta** hacia otro archivo o directorio, similar a un acceso directo de Windows. Si el archivo original se mueve o elimina, el symlink queda "roto" (apuntando a algo que ya no existe). A diferencia de los enlaces duros, los symlinks si pueden cruzar sistemas de archivos y apuntar a directorios.

## Modelo mental

Pensa un enlace duro como dos numeros de telefono distintos que en realidad suenan en la misma casa fisica: no importa cual marques, llegas exactamente al mismo lugar, y la casa solo "desaparece" cuando se cancelan ambos numeros. Un enlace simbolico es como un cartel con una direccion escrita: si la casa a la que apunta esa direccion se muda o se demuele, el cartel sigue ahi pero ya no lleva a ningun lado util.

## Ejemplo base

```bash
ln archivo.txt enlace_duro.txt          # enlace duro: mismo inodo
ln -s /var/log/app.log log_actual        # enlace simbolico: apunta a una ruta

ls -li archivo.txt enlace_duro.txt       # el mismo numero de inodo en ambos
ls -l log_actual                         # muestra la flecha "->" hacia el destino real

rm archivo.txt        # enlace_duro.txt SIGUE teniendo el contenido (otro enlace al mismo inodo)
rm /var/log/app.log    # log_actual queda roto, apuntando a algo que ya no existe
```

## Como se usa en un proyecto real

1. Se usan symlinks para mantener una ruta estable (`/var/www/actual`) que apunta a la version realmente desplegada, cambiando el symlink en cada deploy sin mover archivos reales (patron comun en despliegues sin downtime).
2. Se usan symlinks para acceder facilmente a archivos de configuracion versionados desde su ubicacion esperada por el sistema (por ejemplo, enlazar un dotfile desde un repositorio de configuracion).
3. Se evitan enlaces duros para casos donde se necesita cruzar particiones o apuntar a directorios, dado que no lo soportan; se usa symlink en esos casos.
4. Se verifica con `ls -l` si un symlink esta roto (apunta a algo que ya no existe) como parte de troubleshooting de configuraciones que fallan misteriosamente.
5. Se documenta claramente cuando un archivo importante es en realidad un symlink, para que otros administradores no se confundan sobre donde vive el contenido real.

## Criterio junior

Un junior debe poder crear un enlace simbolico y uno duro, y explicar la diferencia basica entre ambos.

## Criterio mid

Un mid debe diagnosticar un symlink roto y entender por que un enlace duro no puede cruzar particiones ni apuntar a un directorio.

## Criterio senior

Un senior debe diseñar estrategias de despliegue basadas en symlinks (como el patron de "current" apuntando a la ultima version desplegada) para lograr cambios de version practicamente instantaneos y reversibles.

## Errores comunes

- Confundir un enlace simbolico con el archivo real, sorprendiendose cuando el archivo original se mueve y el symlink queda roto.
- Intentar crear un enlace duro hacia un directorio o cruzando particiones distintas, sin saber que eso no esta soportado.
- No verificar si un symlink esta roto al diagnosticar un problema de configuracion que aparentemente "deberia funcionar".
- No aprovechar el patron de symlink para despliegues, complicando innecesariamente el proceso de cambio de version.

## Practica

- [ ] Crea un enlace duro y uno simbolico hacia el mismo archivo, y compara su comportamiento con `ls -li`.
- [ ] Elimina el archivo original y observa que pasa con cada tipo de enlace.
- [ ] Diseña un patron de despliegue con symlink (`current` apuntando a una carpeta versionada) para un proyecto de practica.
- [ ] Identifica un symlink roto en un sistema real o simulado y diagnostica la causa.
- [ ] Anota un caso real donde usarias symlinks para simplificar un proceso de despliegue.

## Preguntas de repaso

- Que diferencia fundamental hay entre un enlace duro y uno simbolico en terminos de como referencian el contenido?
- Por que un enlace duro no puede cruzar particiones ni apuntar a un directorio?
- Que pasa con un symlink si el archivo al que apunta se elimina o se mueve?
- Como usarias un symlink para lograr un cambio de version de despliegue casi instantaneo?
