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

# 01 - Procesos y el Scheduler

## Que vas a aprender

En este capitulo vas a entender que es un **proceso** en Linux, la relacion padre-hijo entre procesos (**fork/exec**), y como el **scheduler** del kernel decide que proceso usa la CPU en cada momento.

## Explicacion

Un proceso es una instancia en ejecucion de un programa, identificado por un **PID** (Process ID) unico. Todo proceso (excepto el primero, PID 1) tiene un **proceso padre** (PPID) que lo creo, tipicamente mediante el patron **fork/exec**: `fork()` crea una copia casi identica del proceso actual, y `exec()` reemplaza esa copia con el nuevo programa a ejecutar. Este es el mecanismo detras de, por ejemplo, que el shell cree un proceso hijo cada vez que ejecutas un comando.

El **scheduler** del kernel decide, entre todos los procesos que quieren usar la CPU, cual la usa en cada momento, dando la ilusion de ejecucion simultanea aunque una CPU fisica solo puede ejecutar una instruccion a la vez (en multiples nucleos, se distribuyen procesos entre ellos, pero el principio de scheduling sigue aplicando dentro de cada nucleo). La **prioridad** de un proceso (ajustable con `nice`) influye en cuanto tiempo de CPU relativo recibe frente a otros procesos.

## Modelo mental

Pensa cada proceso como un empleado trabajando en una oficina con una unica mesa de trabajo compartida (la CPU): el scheduler es el supervisor que decide, turno por turno y muy rapidamente, a quien le toca usar la mesa ahora, dando a todos la sensacion de que trabajan simultaneamente aunque en realidad se turnan.

## Ejemplo base

```bash
ps aux | head              # lista de procesos con detalles (usuario, PID, CPU, memoria)
pstree                     # arbol de procesos mostrando relaciones padre-hijo

echo $$                    # PID del shell actual
sleep 100 &                # lanza un proceso en background
echo $!                    # PID del ultimo proceso lanzado en background

nice -n 10 comando_pesado   # ejecuta con menor prioridad (mas "amable" con otros procesos)
```

## Como se usa en un proyecto real

1. Se usa `pstree` para entender rapidamente la relacion entre un proceso problematico y sus procesos padre/hijo, util para diagnosticar procesos huerfanos o zombies.
2. Se ajusta la prioridad con `nice`/`renice` para tareas pesadas de bajo impacto en la experiencia del usuario (como un backup nocturno) que no deben competir agresivamente por CPU con servicios criticos.
3. Se entiende el mecanismo fork/exec como la base de por que cada comando ejecutado en un script crea un proceso hijo separado, relevante para el capitulo de Scripting con Bash.
4. Se identifica el PID de un proceso especifico antes de enviarle señales (ver siguiente tema), evitando afectar el proceso equivocado.
5. Se conecta este conocimiento con lo visto en el capitulo de fundamentos de Node.js del libro de NestJS (Event Loop), entendiendo que Node.js es, en el fondo, un unico proceso corriendo bajo este mismo modelo de scheduler.

## Criterio junior

Un junior debe poder listar procesos con `ps`, entender que es un PID, y ver la relacion padre-hijo con `pstree`.

## Criterio mid

Un mid debe ajustar prioridades de procesos con `nice`/`renice` para tareas de bajo impacto, y diagnosticar relaciones padre-hijo problematicas.

## Criterio senior

Un senior debe entender el comportamiento del scheduler bajo carga alta para dimensionar correctamente prioridades de servicios criticos versus tareas de mantenimiento, evitando que trabajos de bajo impacto compitan por recursos con servicios de produccion sensibles a latencia.

## Errores comunes

- No usar `nice` para tareas pesadas de bajo impacto, dejando que compitan agresivamente por CPU con servicios criticos del sistema.
- Confundir el PID de un proceso con el de otro similar al enviar señales, afectando el proceso equivocado.
- No entender la relacion fork/exec, dificultando diagnosticar por que un proceso hijo hereda ciertas caracteristicas del padre (como variables de entorno, ver capitulo anterior).
- No usar `pstree` para diagnosticar relaciones complejas entre procesos, perdiendo contexto util para el diagnostico.

## Practica

- [ ] Lista procesos con `ps aux` e identifica el PID y PPID de varios de ellos.
- [ ] Usa `pstree` para visualizar la jerarquia de procesos del sistema actual.
- [ ] Lanza un proceso en background y captura su PID con `$!`.
- [ ] Ejecuta un comando con `nice` ajustando su prioridad.
- [ ] Anota un caso real donde ajustarias la prioridad de una tarea pesada en tu infraestructura.

## Preguntas de repaso

- Que es el patron fork/exec y como se relaciona con la creacion de procesos hijos?
- Que rol cumple el scheduler del kernel entre multiples procesos que compiten por CPU?
- Para que sirve ajustar la prioridad de un proceso con `nice`?
- Como conecta este conocimiento con el modelo de Event Loop de Node.js visto en el libro de NestJS?
