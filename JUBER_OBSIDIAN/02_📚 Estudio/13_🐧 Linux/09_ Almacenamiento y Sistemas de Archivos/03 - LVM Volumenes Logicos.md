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

# 03 - LVM Volumenes Logicos

## Que vas a aprender

En este capitulo vas a aprender **LVM** (Logical Volume Manager), una capa de abstraccion sobre particiones fisicas que permite redimensionar almacenamiento **sin la rigidez** del particionamiento tradicional visto en el primer tema del capitulo.

## Explicacion

LVM introduce tres niveles: **PV** (Physical Volume, un disco o particion fisica real), **VG** (Volume Group, un "pool" que agrupa uno o mas PVs en un unico espacio de almacenamiento combinado), y **LV** (Logical Volume, un volumen "virtual" creado a partir del espacio disponible en un VG, que es lo que finalmente se formatea y monta como si fuera una particion normal). Esta abstraccion permite operaciones que serian muy dificiles con particiones tradicionales: agregar un disco fisico nuevo a un VG existente para extender el espacio disponible sin downtime, o redimensionar un LV (`lvextend`) sin necesitar mover datos manualmente entre particiones.

El caso de uso mas comun es el crecimiento incremental: se crea un LV inicial mas chico de lo que eventualmente se necesitara, y cuando el espacio empieza a agotarse, se extiende (`lvextend` + redimensionar el filesystem con `resize2fs` o `xfs_growfs`) en caliente, sin downtime del servicio que usa ese almacenamiento.

## Modelo mental

Pensa las particiones tradicionales como paredes fijas de concreto en un edificio: cambiarlas requiere demoler y reconstruir. LVM es como paredes movibles modulares: podes agrandar una habitacion (LV) tomando espacio disponible del deposito comun del edificio (VG), sin necesitar romper ninguna pared de concreto real, y sin tener que mudar a los inquilinos mientras se hace el cambio.

## Ejemplo base

```bash
sudo pvcreate /dev/sdb                  # convierte un disco en Physical Volume
sudo vgcreate mi_vg /dev/sdb             # crea un Volume Group con ese PV
sudo lvcreate -L 20G -n mi_lv mi_vg       # crea un Logical Volume de 20GB dentro del VG

sudo mkfs.ext4 /dev/mi_vg/mi_lv          # formatea el LV como cualquier particion
sudo mount /dev/mi_vg/mi_lv /mnt/datos

# Extender el LV cuando el espacio se agota (requiere espacio libre en el VG)
sudo lvextend -L +10G /dev/mi_vg/mi_lv
sudo resize2fs /dev/mi_vg/mi_lv           # extiende el filesystem al nuevo tamaño del LV
```

## Como se usa en un proyecto real

1. Se usa LVM en servidores donde se anticipa la necesidad de crecer almacenamiento con el tiempo, evitando la rigidez de particiones tradicionales.
2. Se crea un LV inicial conservador, sabiendo que se puede extender despues sin downtime, evitando sobre-aprovisionar espacio desde el principio.
3. Se agrega un disco fisico nuevo al VG existente cuando el espacio del pool se agota completamente, extendiendo la capacidad total disponible.
4. Se documenta la estructura de PV/VG/LV de servidores criticos, dado que agrega una capa de abstraccion que un nuevo administrador necesita entender.
5. Se combina LVM con snapshots (una feature adicional de LVM, similar en espiritu a los snapshots de Btrfs vistos en el tema anterior) para backups consistentes sin detener el servicio.

## Criterio junior

Un junior debe entender la relacion PV -> VG -> LV, y poder crear un volumen logico simple siguiendo una guia.

## Criterio mid

Un mid debe extender un LV existente y su filesystem correspondiente sin downtime, y agregar un disco fisico nuevo a un VG existente.

## Criterio senior

Un senior debe diseñar la estrategia de almacenamiento de servidores criticos usando LVM para permitir crecimiento futuro sin downtime, y aprovechar snapshots de LVM para backups consistentes.

## Errores comunes

- Usar particionamiento tradicional rigido en servidores donde se anticipa crecimiento de almacenamiento, complicando innecesariamente futuras expansiones.
- Extender el LV pero olvidar extender tambien el filesystem (`resize2fs`/`xfs_growfs`), dejando el espacio adicional sin usar realmente.
- No documentar la estructura de LVM de un servidor, generando confusion para futuros administradores que no esperan esta capa de abstraccion.
- Sobre-aprovisionar el tamaño inicial de un LV innecesariamente, cuando la ventaja de LVM es precisamente poder extender despues sin downtime.

## Practica

- [ ] Crea un Physical Volume, Volume Group y Logical Volume siguiendo el flujo completo de LVM.
- [ ] Formatea y monta ese LV como usarias cualquier particion normal.
- [ ] Extiende ese LV y su filesystem correspondiente, verificando que el espacio adicional esta disponible.
- [ ] Investiga como funcionan los snapshots de LVM y en que se parecen a los de Btrfs.
- [ ] Documenta la estructura de LVM que usarias para un servidor con necesidad de crecimiento futuro.

## Preguntas de repaso

- Que relacion tienen los conceptos de PV, VG y LV entre si?
- Que ventaja practica da LVM sobre el particionamiento tradicional al momento de crecer el almacenamiento?
- Que paso adicional hay que hacer despues de `lvextend` para que el espacio adicional realmente este disponible en el filesystem?
- En que se parecen conceptualmente los snapshots de LVM a los de Btrfs?
