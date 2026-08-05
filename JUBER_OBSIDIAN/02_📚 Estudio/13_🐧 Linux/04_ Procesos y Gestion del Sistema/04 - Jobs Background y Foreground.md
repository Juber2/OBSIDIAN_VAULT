---
tags:
  - Linux
  - Procesos
  - Linux-Procesos
Creado: 2026-07-28
Relacionado:
  - Procesos y Gestion del Sistema
  - LINUX
---

# 04 - Jobs Background y Foreground

## Que vas a aprender

En este capitulo vas a aprender a manejar procesos en la terminal interactiva: correr comandos en **background** (`&`), traerlos de vuelta al **foreground** (`fg`), pausarlos (`Ctrl+Z`), y la diferencia critica entre un proceso que sobrevive al cierre de la terminal (`nohup`, `disown`) y uno que no.

## Explicacion

Cuando ejecutas un comando normalmente, ocupa el **foreground**: la terminal espera a que termine antes de aceptar otro comando. Agregar `&` al final lo manda a **background**, liberando la terminal para seguir trabajando mientras el comando sigue corriendo. `Ctrl+Z` pausa (suspende) el proceso en foreground actual, `bg` lo reanuda pero en background, y `fg` lo trae de vuelta a foreground. `jobs` lista los procesos en background/pausados de la sesion actual.

Un detalle critico: por defecto, cuando cerras la terminal (o se corta la conexion SSH), los procesos en background de esa sesion reciben SIGHUP y normalmente terminan. `nohup comando &` hace que el proceso ignore SIGHUP y siga corriendo aunque la terminal se cierre; `disown` desvincula un job ya lanzado de la sesion actual con el mismo efecto. Esto es especialmente relevante al trabajar por SSH: sin este cuidado, un proceso largo lanzado por SSH muere si la conexion se corta.

## Modelo mental

Pensa el foreground como tener a alguien trabajando justo frente a vos, esperando que termine antes de darle la siguiente instruccion. El background es enviar a esa persona a otra oficina a seguir trabajando mientras vos atendes otras cosas. Cerrar la terminal sin `nohup` es como si, al apagar las luces de tu oficina, automaticamente se apagaran tambien las luces de todas las oficinas donde mandaste gente a trabajar.

## Ejemplo base

```bash
sleep 300 &          # lanza en background, la terminal queda libre
jobs                 # lista los jobs de esta sesion
fg %1                # trae el job 1 de vuelta a foreground
# Ctrl+Z             # pausa el proceso en foreground actual
bg %1                # reanuda el job 1 pausado, pero en background

nohup ./script_largo.sh &     # sobrevive al cierre de la terminal/sesion SSH
disown -h %1                   # desvincula un job ya lanzado de la sesion actual
```

## Como se usa en un proyecto real

1. Se usa `nohup` (o herramientas mas robustas como `tmux`/`screen`) siempre que se lanza un proceso largo por SSH que debe sobrevivir a un corte de conexion.
2. Se prefiere systemd (ver siguiente tema) para servicios de larga vida en produccion, reservando `nohup`/background manual para tareas puntuales o de administracion.
3. Se usa `jobs` para recordar que procesos en background quedaron corriendo en la sesion actual, evitando perder rastro de tareas lanzadas.
4. Se combina `Ctrl+Z` + `bg` cuando se lanzo algo en foreground por error y se quiere mover a background sin reiniciar el comando desde cero.
5. Se documenta en scripts de deploy si un comando debe correr en background con `nohup` para no bloquear el resto del script.

## Criterio junior

Un junior debe poder lanzar un comando en background con `&`, listarlo con `jobs`, y traerlo de vuelta con `fg`.

## Criterio mid

Un mid debe usar `nohup` o `disown` para asegurar que un proceso largo sobreviva al cierre de una sesion SSH.

## Criterio senior

Un senior debe decidir cuando un proceso realmente necesita correr como job de shell (tareas puntuales de administracion) versus cuando deberia ser un servicio administrado por systemd (procesos de larga vida en produccion).

## Errores comunes

- Lanzar un proceso largo por SSH sin `nohup`, perdiendolo cuando la conexion se corta inesperadamente.
- Confundir procesos en background de la sesion actual con servicios administrados por systemd, usando el mecanismo equivocado para procesos de produccion de larga vida.
- No usar `jobs` para recordar que procesos quedaron corriendo en background, perdiendo rastro de tareas activas.
- Usar `Ctrl+C` cuando en realidad se queria `Ctrl+Z` (pausar en vez de terminar), perdiendo el progreso de un proceso que se queria simplemente mover a background.

## Practica

- [ ] Lanza un comando en background con `&` y verifica que aparece en `jobs`.
- [ ] Pausa un proceso en foreground con `Ctrl+Z` y reanudalo en background con `bg`.
- [ ] Lanza un proceso con `nohup` por una sesion SSH y verifica que sobrevive al cierre de esa sesion.
- [ ] Investiga como `tmux` o `screen` ofrecen una alternativa mas robusta a `nohup` para sesiones de larga duracion.
- [ ] Anota cuando usarias un job de shell vs un servicio systemd para un proceso de larga vida.

## Preguntas de repaso

- Que diferencia hay entre un proceso en foreground y uno en background?
- Que pasa por defecto con los procesos en background de una sesion SSH cuando esa sesion se cierra?
- Para que sirve `nohup` y en que se diferencia de `disown`?
- Cuando conviene usar systemd en vez de simplemente lanzar un proceso en background con `&`?
