---
tags:
  - Linux
  - Almacenamiento
  - Linux-Almacenamiento
Creado: 2026-07-28
Relacionado:
  - Almacenamiento y Sistemas de Archivos
  - LINUX
---

# 04 - Montaje de Discos y fstab

## Que vas a aprender

En este capitulo vas a aprender a **montar** particiones y discos en el arbol de directorios de Linux, tanto de forma temporal (`mount`) como persistente (**`/etc/fstab`**), y a identificar discos de forma confiable usando UUID.

## Explicacion

**Montar** un filesystem significa conectarlo a un punto especifico del arbol de directorios (ver capitulo de FHS), haciendo que su contenido sea accesible en esa ruta. `mount /dev/sdb1 /mnt/datos` es **temporal**: se pierde al reiniciar el sistema, a menos que se agregue una entrada en `/etc/fstab`, el archivo que Linux lee al arrancar para montar automaticamente todos los filesystems que deben estar disponibles desde el inicio.

Un detalle critico de robustez: identificar discos por su nombre de dispositivo (`/dev/sdb1`) es fragil, dado que ese nombre puede cambiar entre reinicios si el orden de deteccion de discos varia (por ejemplo, al agregar o quitar hardware). La practica recomendada es usar el **UUID** (identificador unico e inmutable de cada filesystem) en `/etc/fstab`, garantizando que siempre se monte el disco correcto sin importar en que orden el kernel los detecte.

## Modelo mental

Pensa `mount` temporal como conectar un cable de extension de forma provisoria: funciona mientras nadie lo desconecte, pero no sobrevive un "reinicio de la casa". `/etc/fstab` es como dejar esa conexion cableada permanentemente en la instalacion electrica: se reconecta automaticamente cada vez que se enciende la luz general (boot), sin necesitar que alguien lo haga manualmente cada vez.

## Ejemplo base

```bash
sudo mount /dev/sdb1 /mnt/datos      # montaje temporal, se pierde al reiniciar

blkid                                 # muestra el UUID de cada dispositivo/particion
```

```
# /etc/fstab: monta automaticamente en cada boot, usando UUID (no /dev/sdX, que puede cambiar)
UUID=1234-5678-90ab-cdef  /mnt/datos  ext4  defaults  0  2
```

```bash
sudo mount -a          # aplica /etc/fstab sin reiniciar, util para verificar que no hay errores de sintaxis
```

## Como se usa en un proyecto real

1. Se agrega toda particion adicional (discos de datos, almacenamiento externo) a `/etc/fstab` para que este disponible automaticamente despues de cualquier reinicio del servidor.
2. Se usa siempre UUID en vez de `/dev/sdX` en `/etc/fstab`, evitando el riesgo de que el sistema monte el disco equivocado si el orden de deteccion cambia.
3. Se verifica un cambio en `/etc/fstab` con `sudo mount -a` antes de reiniciar, dado que un error de sintaxis en ese archivo puede impedir que el sistema arranque correctamente.
4. Se documenta el proposito de cada entrada en `/etc/fstab` de un servidor, especialmente en configuraciones con multiples discos.
5. Se conecta este conocimiento con el capitulo de Volumenes del libro de Docker: los volumenes de Docker en el fondo se apoyan en el mismo sistema de montaje de Linux, solo gestionado automaticamente por el daemon.

## Criterio junior

Un junior debe poder montar un disco temporalmente con `mount`, y entender que `/etc/fstab` es necesario para que el montaje persista entre reinicios.

## Criterio mid

Un mid debe agregar entradas correctas en `/etc/fstab` usando UUID (no nombre de dispositivo), y verificar cambios con `mount -a` antes de confiar en un reinicio.

## Criterio senior

Un senior debe diseñar la configuracion de montaje de servidores con multiples discos, documentando el proposito de cada montaje, y evitar errores de `/etc/fstab` que puedan impedir el arranque de un servidor de produccion.

## Errores comunes

- Usar `/dev/sdX` en `/etc/fstab` en vez de UUID, arriesgando que el sistema monte el disco equivocado si el orden de deteccion cambia entre reinicios.
- No verificar un cambio en `/etc/fstab` con `mount -a` antes de confiar en un reinicio, arriesgando que el servidor no arranque correctamente por un error de sintaxis.
- Montar un disco solo temporalmente con `mount`, olvidando que ese montaje se perdera en el proximo reinicio si no se agrega a `/etc/fstab`.
- No documentar el proposito de cada entrada en `/etc/fstab`, dificultando el mantenimiento de servidores con configuraciones de almacenamiento complejas.

## Practica

- [ ] Monta un disco o particion temporalmente con `mount` y verifica su contenido.
- [ ] Encuentra el UUID de ese dispositivo con `blkid`.
- [ ] Agrega una entrada correcta en `/etc/fstab` usando ese UUID.
- [ ] Verifica el cambio con `mount -a` antes de considerar reiniciar el sistema.
- [ ] Documenta el proposito de cada entrada de `/etc/fstab` en un sistema real que administres.

## Preguntas de repaso

- Que diferencia hay entre montar un disco con `mount` directamente y agregarlo a `/etc/fstab`?
- Por que usar UUID en `/etc/fstab` es mas seguro que usar el nombre de dispositivo `/dev/sdX`?
- Que verificacion harias despues de modificar `/etc/fstab`, antes de reiniciar el servidor?
- Como se relaciona el sistema de montaje de Linux con los volumenes de Docker vistos en el libro correspondiente?
