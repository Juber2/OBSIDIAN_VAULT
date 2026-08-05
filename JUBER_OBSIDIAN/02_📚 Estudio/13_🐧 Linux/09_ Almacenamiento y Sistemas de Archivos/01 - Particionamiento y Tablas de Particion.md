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

# 01 - Particionamiento y Tablas de Particion

## Que vas a aprender

En este capitulo vas a entender como se organiza un disco fisico en Linux: **particiones**, las dos tablas de particion principales (**MBR** y **GPT**), y las herramientas para crear y gestionar particiones.

## Explicacion

Un disco fisico puede dividirse en **particiones**, cada una comportandose como un disco independiente que puede formatearse con su propio sistema de archivos y montarse en un punto especifico del arbol de directorios (ver capitulo de FHS). Particionar permite, por ejemplo, separar el sistema operativo de los datos de usuario, o dedicar una particion completa a `/var` para que logs excesivos no puedan llenar el disco raiz completo.

**MBR** (Master Boot Record) es el esquema clasico, limitado a 4 particiones primarias y discos de hasta 2TB. **GPT** (GUID Partition Table) es el estandar moderno, sin esas limitaciones practicas (soporta muchas mas particiones y discos mucho mas grandes), y es el recomendado para cualquier instalacion nueva salvo necesidad especifica de compatibilidad con sistemas muy antiguos. Herramientas como `fdisk` (tradicional, orientado a MBR) y `parted`/`gdisk` (con mejor soporte para GPT) permiten crear, modificar y eliminar particiones.

## Modelo mental

Pensa un disco fisico como un terreno grande sin dividir, y particionar como trazar los limites de lotes individuales dentro de ese terreno: cada lote (particion) puede construirse y usarse de forma independiente, con su propia "direccion" (punto de montaje) dentro del sistema general de direcciones del vecindario (el arbol de directorios).

## Ejemplo base

```bash
lsblk                          # lista discos y particiones existentes, con jerarquia visual
sudo fdisk -l                   # informacion detallada de particiones (requiere sudo)

sudo parted /dev/sdb            # herramienta interactiva para crear particiones GPT
# (mklabel gpt, mkpart primary ext4 0% 100%, print, quit)

sudo mkfs.ext4 /dev/sdb1         # formatea la particion recien creada con ext4
```

## Como se usa en un proyecto real

1. Se usa `lsblk` como primera herramienta para entender la estructura de discos y particiones de un servidor antes de hacer cualquier cambio.
2. Se elige GPT por defecto para instalaciones nuevas, reservando MBR solo para compatibilidad con sistemas legacy que lo requieran explicitamente.
3. Se separa `/var` en una particion propia en servidores donde los logs pueden crecer significativamente, evitando que llenen el disco raiz y tumben todo el sistema.
4. Se documenta el esquema de particionamiento de servidores criticos, incluyendo el proposito de cada particion.
5. Se hace siempre un respaldo antes de modificar particiones existentes, dado que operaciones de particionamiento incorrectas pueden causar perdida total de datos.

## Criterio junior

Un junior debe poder listar discos y particiones con `lsblk`, y entender la diferencia conceptual basica entre MBR y GPT.

## Criterio mid

Un mid debe crear particiones nuevas con `parted` o `fdisk`, y decidir un esquema de particionamiento razonable para un servidor nuevo (separando `/var`, por ejemplo).

## Criterio senior

Un senior debe diseñar el esquema de particionamiento de servidores criticos considerando crecimiento futuro, aislamiento de fallos (un directorio lleno no debe tumbar todo el sistema), y estrategias de backup por particion.

## Errores comunes

- Modificar particiones existentes con datos importantes sin un respaldo previo, arriesgando perdida total de datos ante un error.
- No separar `/var` (o directorios de crecimiento impredecible) en servidores donde los logs pueden crecer significativamente, arriesgando que llenen el disco raiz completo.
- Usar MBR en una instalacion nueva sin necesidad real de esa compatibilidad legacy, perdiendo las ventajas de GPT.
- No documentar el esquema de particionamiento de un servidor, dificultando el mantenimiento futuro.

## Practica

- [ ] Lista los discos y particiones de tu sistema con `lsblk`.
- [ ] Investiga si tu sistema usa MBR o GPT con `fdisk -l` o `parted print`.
- [ ] Si tenes un disco de practica disponible, crea una particion nueva con GPT.
- [ ] Formatea esa particion con `mkfs.ext4` (siguiente capitulo profundiza en sistemas de archivos).
- [ ] Diseña un esquema de particionamiento propio para un servidor de produccion hipotetico.

## Preguntas de repaso

- Que ventaja tiene GPT sobre MBR para instalaciones modernas?
- Por que separar `/var` en su propia particion puede prevenir un incidente de disco lleno?
- Que herramienta usarias para explorar rapidamente la estructura de discos de un sistema?
- Que precaucion tomarias siempre antes de modificar particiones existentes con datos?
