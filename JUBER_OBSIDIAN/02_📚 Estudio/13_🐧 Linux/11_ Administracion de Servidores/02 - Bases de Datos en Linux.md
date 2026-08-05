---
tags:
  - Linux
  - Administracion
  - Linux-Servidores
Creado: 2026-07-28
Relacionado:
  - Administracion de Servidores
  - LINUX
---

# 02 - Bases de Datos en Linux

## Que vas a aprender

En este capitulo vas a aprender las consideraciones de administracion de sistema al correr bases de datos (Postgres, MySQL) directamente sobre Linux (sin container): instalacion, configuracion basica de performance, y separacion de almacenamiento, conectando con lo ya visto en el libro de NestJS y Docker.

## Explicacion

Una base de datos instalada directamente sobre Linux (en vez de en un container, como se vio en el libro de Docker) sigue las mismas consideraciones de administracion de sistema vistas en este libro: el proceso corre como un usuario dedicado (aplicando el principio de menor privilegio ya visto), sus datos viven en un directorio especifico que idealmente esta en su propia particion o volumen LVM (ver capitulo de Almacenamiento), y se administra como un servicio systemd con reinicio automatico.

Configuraciones basicas de performance a nivel de sistema operativo que afectan directamente a una base de datos: los limites de archivos abiertos (`ulimit`, relevante porque una base de datos con muchas conexiones concurrentes puede agotar el limite por defecto del sistema), la configuracion de memoria compartida del kernel (relevante para Postgres especificamente), y el scheduler de I/O del disco donde viven los datos.

## Modelo mental

Pensa correr una base de datos directamente sobre Linux como ser dueño de tu propio restaurante con cocina propia, en vez de alquilar un espacio de cocina compartido (container): tenes control total sobre cada configuracion, pero tambien la responsabilidad completa de mantener todo funcionando correctamente, desde los permisos de acceso hasta el espacio de almacenamiento dedicado.

## Ejemplo base

```bash
sudo apt install postgresql
sudo systemctl status postgresql        # administrado como servicio systemd

# Verificar el usuario dedicado bajo el que corre
ps aux | grep postgres

# Limites de archivos abiertos, relevante para conexiones concurrentes
ulimit -n                                # limite actual de la sesion
cat /etc/security/limits.conf             # limites persistentes por usuario/servicio

sudo -u postgres psql -c "SHOW data_directory;"   # donde vive fisicamente los datos
```

## Como se usa en un proyecto real

1. Se ubica el directorio de datos de la base de datos en una particion o volumen LVM separado (ver capitulo de Almacenamiento), evitando que el crecimiento de datos afecte al disco raiz del sistema.
2. Se ajustan limites de archivos abiertos (`ulimit`, `/etc/security/limits.conf`) segun la cantidad de conexiones concurrentes esperadas, evitando errores de "too many open files" bajo carga.
3. Se administra la base de datos como servicio systemd con `Restart=on-failure`, aplicando el mismo patron ya visto en el capitulo de Procesos de este libro.
4. Se evalua conscientemente correr la base de datos directamente sobre Linux versus en un container (ver capitulo de Bases de Datos en Containers del libro de Docker), segun necesidades de performance y control.
5. Se conecta la configuracion de conexion de la base de datos con lo visto en el capitulo de Base de Datos del libro de NestJS (TypeORM/Prisma), asegurando consistencia entre la configuracion de infraestructura y de aplicacion.

## Criterio junior

Un junior debe poder instalar una base de datos con el gestor de paquetes del sistema y verificar que corre como servicio administrado.

## Criterio mid

Un mid debe ajustar limites de sistema (`ulimit`) apropiados para la carga esperada de conexiones, y ubicar el directorio de datos en almacenamiento dedicado.

## Criterio senior

Un senior debe decidir con criterio cuando una base de datos corre directamente sobre Linux versus en un container (o servicio gestionado, ver libro de NestJS), y optimizar la configuracion de sistema operativo especificamente para el motor de base de datos usado.

## Errores comunes

- No ajustar limites de archivos abiertos, causando errores de conexion bajo carga alta que un limite por defecto conservador no anticipa.
- Dejar el directorio de datos en el disco raiz sin particion o volumen dedicado, arriesgando que el crecimiento de la base de datos afecte al sistema completo.
- No administrar la base de datos como servicio systemd apropiadamente configurado, perdiendo reinicio automatico ante fallos.
- No coordinar la configuracion de infraestructura de la base de datos con la configuracion de conexion de la aplicacion (ver libro de NestJS).

## Practica

- [ ] Instala una base de datos (Postgres o MySQL) con el gestor de paquetes de tu distribucion.
- [ ] Verifica que corre como servicio systemd administrado, con el usuario dedicado correcto.
- [ ] Revisa y ajusta limites de archivos abiertos apropiados para la carga esperada.
- [ ] Verifica donde vive fisicamente el directorio de datos y evalua si deberia estar en almacenamiento dedicado.
- [ ] Compara conceptualmente correr esta base de datos directamente vs en un container Docker.

## Preguntas de repaso

- Por que conviene ubicar el directorio de datos de una base de datos en una particion o volumen separado del disco raiz?
- Que problema previene ajustar correctamente los limites de archivos abiertos para una base de datos con muchas conexiones?
- Que factores considerarias para decidir entre correr una base de datos directamente sobre Linux o en un container?
- Como se conecta la configuracion de infraestructura de una base de datos con lo visto en el capitulo correspondiente del libro de NestJS?
