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

# 02 - Kernel Espacio de Usuario y Arquitectura

## Que vas a aprender

En este capitulo vas a entender la separacion entre **espacio de kernel** y **espacio de usuario**, y los subsistemas principales del kernel Linux (gestion de procesos, memoria, filesystem, red) que ya viste aplicados de forma practica en el libro de Docker (namespaces, cgroups).

## Explicacion

El kernel corre en un modo privilegiado de la CPU (**kernel space**), con acceso directo al hardware, mientras que todos los programas normales (el shell, un navegador, tu API en Node) corren en **user space**, con acceso restringido que debe pasar por **syscalls** (llamadas al sistema) cuando necesitan algo del kernel: leer un archivo, abrir una conexion de red, crear un proceso. Esta separacion es la base de la estabilidad y seguridad de Linux: un programa de usuario que falla no puede, en condiciones normales, tumbar el kernel completo.

Internamente, el kernel se organiza en subsistemas: el **scheduler** decide que proceso usa la CPU en cada momento, el **gestor de memoria** asigna y protege memoria entre procesos, la capa de **VFS** (Virtual File System) abstrae distintos sistemas de archivos bajo una interfaz comun, y la **pila de red** implementa TCP/IP. Estos son precisamente los mecanismos (namespaces, cgroups) que Docker usa para aislar containers, ya vistos en profundidad en el libro correspondiente.

## Modelo mental

Pensa el kernel como el personal de seguridad y mantenimiento de un edificio de oficinas: tiene las llaves maestras de todo (acceso total al hardware), mientras que los empleados de cada oficina (procesos de usuario) solo pueden pedir cosas especificas al personal de seguridad (syscalls) en vez de acceder directamente a areas restringidas del edificio.

## Ejemplo base

```bash
strace -c ls /tmp        # cuenta cuantas syscalls hizo un comando simple
cat /proc/cpuinfo         # informacion expuesta por el kernel sobre el hardware
cat /proc/meminfo         # informacion de memoria expuesta por el kernel
lsmod                     # modulos del kernel actualmente cargados
```

## Como se usa en un proyecto real

1. Se usa `/proc` y `/sys` (interfaces virtuales expuestas por el kernel) como fuente de verdad para inspeccionar el estado real del sistema, en vez de asumir o adivinar.
2. Se investiga con `strace` cuando una aplicacion se comporta de forma inesperada relacionada con archivos, red o permisos, viendo exactamente que syscalls falla y por que.
3. Se entiende que un crash de una aplicacion de usuario normalmente no afecta la estabilidad del kernel ni de otros procesos, gracias a esta separacion de espacios.
4. Se conecta este conocimiento con lo aprendido en Docker: namespaces y cgroups (capitulos de Redes y Seguridad del libro de Docker) son features del kernel que aislan procesos usando estos mismos mecanismos.
5. Se investiga la version y configuracion del kernel al diagnosticar comportamientos especificos de una version particular en produccion.

## Criterio junior

Un junior debe entender la diferencia conceptual entre espacio de kernel y espacio de usuario, y saber que un programa normal no puede acceder directo al hardware sin pasar por el kernel.

## Criterio mid

Un mid debe usar `/proc`, `/sys` y herramientas como `strace` para diagnosticar el comportamiento real de un proceso a nivel de syscalls.

## Criterio senior

Un senior debe entender como estos mecanismos de bajo nivel (namespaces, cgroups) sustentan tecnologias de mayor nivel como containers, y usar ese conocimiento para diagnosticar problemas complejos de aislamiento o performance en produccion.

## Errores comunes

- Asumir que un crash de una aplicacion puede tumbar el kernel completo, sin entender la proteccion que da la separacion de espacios.
- No usar `/proc` y `/sys` como fuente de verdad del estado del sistema, confiando solo en herramientas de mas alto nivel que a veces abstraen demasiado.
- No conocer `strace` como herramienta de diagnostico cuando un problema parece relacionado con archivos, permisos o red a bajo nivel.
- No conectar este conocimiento fundamental con tecnologias que ya se usan a diario, como containers, perdiendo la oportunidad de diagnosticar problemas mas profundos.

## Practica

- [ ] Usa `strace` sobre un comando simple y cuenta cuantas syscalls realiza.
- [ ] Explora `/proc/cpuinfo` y `/proc/meminfo` para entender que informacion expone el kernel.
- [ ] Lista los modulos del kernel cargados con `lsmod`.
- [ ] Investiga como namespaces y cgroups (vistos en el libro de Docker) usan estos mecanismos del kernel.
- [ ] Anota un caso donde `strace` te ayudaria a diagnosticar un problema real.

## Preguntas de repaso

- Que diferencia hay entre espacio de kernel y espacio de usuario?
- Que es una syscall y por que un programa de usuario la necesita para acceder a hardware?
- Que informacion util exponen las interfaces virtuales `/proc` y `/sys`?
- Como se conectan namespaces y cgroups (usados por Docker) con estos conceptos fundamentales del kernel?
