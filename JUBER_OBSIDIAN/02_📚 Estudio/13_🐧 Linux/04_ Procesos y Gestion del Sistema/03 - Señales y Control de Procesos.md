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

# 03 - Señales y Control de Procesos

## Que vas a aprender

En este capitulo vas a profundizar en **señales** de Unix/Linux, el mecanismo de comunicacion basico entre procesos y el kernel: la diferencia critica entre **SIGTERM** y **SIGKILL** (ya mencionada en el libro de Docker), y otras señales importantes.

## Explicacion

Una señal es una notificacion asincrona enviada a un proceso, que puede manejar de forma personalizada (registrando un handler), ignorar, o dejar que el kernel aplique su comportamiento por defecto. **SIGTERM** (señal 15, la que envia `kill` por defecto) pide a un proceso que termine de forma **graceful**, dandole la oportunidad de cerrar conexiones, guardar estado, o limpiar recursos antes de salir. **SIGKILL** (señal 9) es un corte inmediato e inevitable: el proceso no puede capturarla ni ignorarla, termina de inmediato sin oportunidad de limpieza, arriesgando corrupcion de datos si estaba a mitad de una operacion critica.

Otras señales relevantes: **SIGHUP** (historicamente "colgar", usada convencionalmente por muchos daemons para recargar configuracion sin reiniciar completamente), **SIGINT** (lo que envia `Ctrl+C` desde la terminal), **SIGSTOP/SIGCONT** (pausar y reanudar un proceso, similar a `docker pause`/`unpause` visto en el libro de Docker).

## Modelo mental

Pensa SIGTERM como pedirle educadamente a alguien que se vaya, dandole tiempo de agarrar sus cosas y despedirse (graceful shutdown). SIGKILL es como cortarle la luz de golpe sin aviso: se va inmediatamente, pero si estaba en medio de algo importante, eso queda a medio terminar sin ninguna posibilidad de reaccionar.

## Ejemplo base

```bash
kill 1234                  # envia SIGTERM (default) al proceso con PID 1234
kill -9 1234                # envia SIGKILL, corte inmediato sin oportunidad de limpieza
kill -HUP 1234               # muchos daemons interpretan esto como "recarga tu configuracion"

killall nombre_proceso        # envia señal a todos los procesos que coincidan por nombre
pkill -f "patron de comando"  # mata procesos cuyo comando completo coincida con un patron

trap 'echo "Recibi SIGTERM, limpiando..."; exit 0' TERM   # manejar SIGTERM en un script bash
```

## Como se usa en un proyecto real

1. Se usa siempre `kill` simple (SIGTERM) como primera opcion para detener un proceso, reservando `kill -9` (SIGKILL) solo para procesos realmente colgados que no responden.
2. Se diseñan aplicaciones (y scripts) que manejan SIGTERM correctamente, cerrando conexiones de base de datos y terminando operaciones en curso antes de salir, evitando el mismo problema visto en el capitulo de Ciclo de Vida de Docker.
3. Se usa SIGHUP convencionalmente en daemons propios para soportar recarga de configuracion sin downtime completo, si la aplicacion lo implementa.
4. Se usa `pkill -f` con cuidado, verificando el patron exacto antes de ejecutarlo, dado que puede coincidir con mas procesos de los esperados si el patron es demasiado generico.
5. Se documenta en scripts de administracion que señal se espera que maneje cada servicio propio, facilitando el trabajo de quien opera esos servicios despues.

## Criterio junior

Un junior debe poder enviar SIGTERM y SIGKILL a un proceso con `kill`, y entender por que SIGTERM deberia intentarse siempre primero.

## Criterio mid

Un mid debe usar `pkill`/`killall` de forma segura verificando el patron antes de ejecutar, y diagnosticar cuando un proceso no responde a SIGTERM.

## Criterio senior

Un senior debe diseñar aplicaciones y servicios que manejen correctamente señales del sistema operativo para shutdown graceful, y establecer convenciones de equipo sobre el uso de SIGHUP para recarga de configuracion sin downtime.

## Errores comunes

- Usar `kill -9` como primera opcion por costumbre, sin dar oportunidad de un cierre limpio con SIGTERM primero.
- Usar `pkill -f` con un patron demasiado generico, matando procesos no relacionados que coincidieron accidentalmente.
- No implementar manejo de SIGTERM en aplicaciones propias, perdiendo la oportunidad de un shutdown graceful cuando se necesita detener el servicio.
- Confundir el comportamiento de distintas señales, esperando que SIGKILL pueda ser capturado o ignorado como otras señales.

## Practica

- [ ] Envia SIGTERM a un proceso de prueba y observa su comportamiento de cierre.
- [ ] Compara el efecto de SIGKILL sobre el mismo tipo de proceso.
- [ ] Escribe un script Bash simple que capture SIGTERM con `trap` y haga limpieza antes de salir.
- [ ] Usa `pkill -f` con un patron especifico, verificando antes que procesos coincidirian.
- [ ] Anota que señal implementarias en un servicio propio para soportar recarga de configuracion.

## Preguntas de repaso

- Por que SIGTERM deberia intentarse siempre antes que SIGKILL?
- Que diferencia de comportamiento hay entre una señal que un proceso puede capturar y SIGKILL que no puede?
- Que uso convencional le dan muchos daemons a SIGHUP?
- Que riesgo tiene usar `pkill -f` con un patron demasiado generico?
