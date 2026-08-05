---
tags:
  - Linux
  - Sistemas
  - Linux-Fundamentos
Creado: 2026-07-28
Relacionado:
  - Fundamentos del Sistema
  - LINUX
---

# 03 - Filesystem Hierarchy Standard

## Que vas a aprender

En este capitulo vas a aprender el **Filesystem Hierarchy Standard (FHS)**: la estructura de directorios estandarizada que casi todas las distribuciones Linux respetan, y para que sirve cada directorio principal.

## Explicacion

A diferencia de Windows (con letras de unidad como `C:\`), Linux tiene un **unico arbol de directorios** que empieza en la raiz `/`, donde todo (incluyendo otros discos montados) cuelga de ahi. El FHS define convenciones sobre que va en cada lugar: `/etc` para configuracion del sistema, `/var` para datos que cambian frecuentemente (logs, cache, bases de datos), `/home` para los directorios personales de usuarios, `/usr` para programas y sus recursos instalados por el sistema de paquetes, `/tmp` para archivos temporales que pueden borrarse en cualquier reinicio, `/opt` para software de terceros instalado manualmente fuera del gestor de paquetes.

Respetar esta estructura (en vez de instalar cosas donde "parece comodo") es lo que permite que herramientas de administracion, scripts de backup, y otros administradores puedan predecir donde encontrar algo en cualquier sistema Linux, sin importar la distribucion especifica.

## Modelo mental

Pensa el FHS como el codigo de urbanismo de una ciudad: define que zona es residencial (`/home`), cual es comercial (`/usr`), cual es de servicios publicos temporales (`/tmp`), y cual es de archivo de documentos oficiales que cambian (`/var`). Cualquier ciudadano (administrador) que llega a una ciudad nueva (servidor nuevo) puede orientarse rapido porque el codigo de urbanismo es el mismo en todas las ciudades del pais.

## Ejemplo base

```bash
ls /                # ver la raiz y sus directorios principales
du -sh /var/log/*    # ver que esta ocupando espacio dentro de /var/log
df -h /              # espacio disponible en el filesystem raiz
```

```
/etc      -> configuracion del sistema (nginx.conf, passwd, hosts)
/var      -> datos variables: logs, cache, spool, bases de datos locales
/home     -> directorios personales de usuarios (/home/juber)
/usr      -> binarios, librerias y recursos de software instalado
/tmp      -> archivos temporales, puede vaciarse en cada reinicio
/opt      -> software de terceros instalado manualmente
/root     -> directorio personal del usuario root (distinto de /)
```

## Como se usa en un proyecto real

1. Se guarda la configuracion de servicios propios en `/etc`, siguiendo la convencion que cualquier otro administrador esperaria encontrar.
2. Se almacenan logs y datos que crecen con el tiempo en `/var`, monitoreando ese espacio especificamente dado que es el mas propenso a llenarse en un servidor de larga duracion.
3. Se evita guardar datos importantes en `/tmp`, dado que puede limpiarse automaticamente en cada reinicio del sistema segun la configuracion de la distribucion.
4. Se instala software que no viene del gestor de paquetes oficial en `/opt`, manteniendolo claramente separado del software gestionado por el sistema.
5. Se documenta cualquier desviacion deliberada de esta convencion en un proyecto especifico, para que otros administradores no se sorprendan.

## Criterio junior

Un junior debe conocer el proposito de los directorios principales (`/etc`, `/var`, `/home`, `/usr`, `/tmp`) y poder ubicar archivos de configuracion o logs comunes.

## Criterio mid

Un mid debe diagnosticar problemas de espacio en disco sabiendo donde buscar (tipicamente `/var/log`), y estructurar la instalacion de software propio siguiendo la convencion del FHS.

## Criterio senior

Un senior debe diseñar la estrategia de particionamiento de un servidor (ver capitulo de Almacenamiento) tomando en cuenta el FHS, por ejemplo dedicando una particion separada a `/var` en servidores donde los logs pueden crecer significativamente.

## Errores comunes

- Guardar datos criticos en `/tmp`, perdiendolos ante un reinicio que limpia ese directorio.
- No monitorear el crecimiento de `/var/log`, dejando que los logs llenen el disco del sistema con el tiempo.
- Instalar software de formas no convencionales dispersas por el sistema, dificultando su mantenimiento y desinstalacion posterior.
- No conocer donde buscar configuracion o logs de un servicio nuevo, perdiendo tiempo de diagnostico innecesario.

## Practica

- [ ] Explora la raiz del sistema (`ls /`) e identifica el proposito de cada directorio principal.
- [ ] Revisa que esta ocupando espacio dentro de `/var/log` en un sistema real o de practica.
- [ ] Ubica el archivo de configuracion de un servicio instalado (por ejemplo, Nginx en `/etc/nginx`).
- [ ] Verifica el espacio disponible del filesystem raiz con `df -h`.
- [ ] Anota donde instalarias software de terceros que no viene de un gestor de paquetes.

## Preguntas de repaso

- Que diferencia hay entre `/usr` y `/opt` en cuanto al origen del software instalado?
- Por que `/var` es el directorio que mas hay que monitorear en un servidor de larga duracion?
- Que riesgo tiene guardar datos importantes en `/tmp`?
- Como te orientarias rapidamente en un servidor Linux nuevo gracias al FHS?
