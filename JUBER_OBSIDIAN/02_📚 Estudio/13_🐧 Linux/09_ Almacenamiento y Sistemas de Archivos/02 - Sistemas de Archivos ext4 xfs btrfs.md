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

# 02 - Sistemas de Archivos ext4 xfs btrfs

## Que vas a aprender

En este capitulo vas a comparar los sistemas de archivos mas usados en Linux: **ext4** (el estandar tradicional), **XFS** (orientado a alto rendimiento con archivos grandes), y **Btrfs** (moderno, con features avanzadas como snapshots), con criterios para elegir cada uno.

## Explicacion

Un sistema de archivos define como se organizan y acceden los datos dentro de una particion (a diferencia del particionamiento, que define los limites del espacio). **ext4** es el sistema de archivos por defecto en la mayoria de distribuciones, maduro, estable, y con buen soporte universal: la eleccion segura por defecto para la mayoria de casos. **XFS** (default en RHEL/CentOS) esta optimizado para archivos grandes y alto paralelismo de I/O, comun en servidores de bases de datos o almacenamiento de gran volumen.

**Btrfs** ofrece features modernas que ext4/XFS no tienen nativamente: **snapshots** (capturas instantaneas del estado del filesystem, extremadamente utiles antes de una actualizacion riesgosa del sistema, permitiendo revertir facilmente), compresion transparente, y checksums de datos para detectar corrupcion silenciosa. A cambio, es mas complejo de administrar y, historicamente, algo menos maduro/estable que ext4 en ciertos escenarios de carga extrema.

## Modelo mental

Pensa ext4 como un auto sedan confiable y bien probado, adecuado para la gran mayoria de trayectos diarios sin sorpresas. XFS es como un camion optimizado especificamente para cargar objetos muy pesados y grandes eficientemente. Btrfs es como un vehiculo con tecnologia avanzada (camara de retroceso, grabacion continua): mas capacidades utiles, pero tambien mas sistemas que pueden fallar y requieren mas conocimiento para aprovechar correctamente.

## Ejemplo base

```bash
mkfs.ext4 /dev/sdb1        # formatea con ext4, la opcion segura por defecto
mkfs.xfs /dev/sdb1          # formatea con XFS, mejor para archivos grandes/alto I/O
mkfs.btrfs /dev/sdb1         # formatea con Btrfs, para aprovechar snapshots y compresion

df -T                        # muestra el tipo de filesystem de cada particion montada

# Btrfs: crear un snapshot antes de una actualizacion riesgosa
btrfs subvolume snapshot /mnt/datos /mnt/datos-snapshot-2026-07-28
```

## Como se usa en un proyecto real

1. Se usa ext4 por defecto salvo una razon especifica que justifique otra opcion, dado que es la eleccion mas ampliamente probada y compatible.
2. Se elige XFS para servidores dedicados a bases de datos o almacenamiento de archivos muy grandes, aprovechando su optimizacion de I/O paralelo.
3. Se evalua Btrfs cuando snapshots faciles de crear/revertir son valiosos, como antes de actualizaciones de sistema riesgosas o para backups incrementales eficientes.
4. Se documenta el sistema de archivos elegido para cada particion critica y la justificacion de esa eleccion.
5. Se conecta el concepto de snapshots de Btrfs con lo ya visto sobre backup de volumenes en el libro de Docker: ambos resuelven el mismo problema (poder revertir a un estado anterior) en capas distintas del stack.

## Criterio junior

Un junior debe conocer las tres opciones (ext4, XFS, Btrfs) y saber que ext4 es la eleccion segura por defecto en la mayoria de casos.

## Criterio mid

Un mid debe elegir XFS o Btrfs cuando el caso de uso especifico lo justifique (archivos grandes, necesidad de snapshots), documentando la razon.

## Criterio senior

Un senior debe diseñar la estrategia de sistemas de archivos de toda la infraestructura segun el tipo de carga de cada servidor, aprovechando features avanzadas como snapshots de Btrfs para procesos de actualizacion mas seguros.

## Errores comunes

- Elegir un sistema de archivos exotico sin necesidad real, agregando complejidad de administracion sin beneficio proporcional.
- No aprovechar snapshots de Btrfs antes de operaciones riesgosas cuando ese sistema de archivos ya esta en uso, perdiendo una red de seguridad disponible.
- No documentar por que se eligio un sistema de archivos especifico para una particion, dificultando decisiones futuras de mantenimiento.
- Asumir que todos los sistemas de archivos se comportan igual en cuanto a performance, sin considerar el tipo de carga especifico del servidor.

## Practica

- [ ] Verifica que sistema de archivos usa cada particion montada en tu sistema con `df -T`.
- [ ] Formatea una particion de practica con ext4 y otra con XFS, comparando el proceso.
- [ ] Si tenes Btrfs disponible, crea un snapshot y practica revertir a el.
- [ ] Investiga que sistema de archivos usa por defecto tu distribucion actual.
- [ ] Anota un caso real donde elegirias XFS o Btrfs sobre ext4.

## Preguntas de repaso

- Por que ext4 sigue siendo la eleccion por defecto segura para la mayoria de casos?
- Que ventaja especifica tiene XFS para servidores con archivos muy grandes?
- Que feature de Btrfs es especialmente util antes de una actualizacion riesgosa del sistema?
- Como se relaciona el concepto de snapshot de Btrfs con el backup de volumenes visto en Docker?
