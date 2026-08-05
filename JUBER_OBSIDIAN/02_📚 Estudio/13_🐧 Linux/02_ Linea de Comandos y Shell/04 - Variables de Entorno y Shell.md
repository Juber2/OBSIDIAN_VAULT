---
tags:
  - Linux
  - Bash
  - Linux-Shell
Creado: 2026-07-28
Relacionado:
  - Linea de Comandos y Shell
  - LINUX
---

# 04 - Variables de Entorno y Shell

## Que vas a aprender

En este capitulo vas a entender la diferencia entre **variables de shell** y **variables de entorno**, como se exportan y heredan entre procesos, y las variables mas importantes del sistema (`PATH`, `HOME`, `USER`).

## Explicacion

Una **variable de shell** existe solo dentro de la sesion actual del shell (`variable=valor`); una **variable de entorno** (`export variable=valor`) se hereda ademas por cualquier proceso hijo que ese shell inicie, siendo la forma en que Node, Python, o cualquier programa lanzado desde la terminal puede leer configuracion sin necesitar argumentos explicitos. Esto conecta directamente con el capitulo de `ConfigModule` en NestJS: las variables de entorno que la aplicacion lee con `process.env` son, en el fondo, exactamente este mecanismo del shell.

`PATH` es una de las variables de entorno mas importantes: contiene una lista de directorios (separados por `:`) donde el shell busca ejecutables cuando escribis un comando sin ruta completa. Si un programa instalado no se encuentra al ejecutarlo, casi siempre es porque su directorio no esta en `PATH`. Los archivos de configuracion del shell (`.bashrc`, `.bash_profile`, `.profile`) son donde se definen estas variables de forma persistente entre sesiones.

## Modelo mental

Pensa una variable de shell como una nota pegada en tu propio escritorio (solo vos la ves), y una variable de entorno exportada como una nota que se fotocopia y se le entrega automaticamente a cualquier ayudante (proceso hijo) que contrates desde tu escritorio: ellos reciben esa informacion sin que se las tengas que decir explicitamente cada vez.

## Ejemplo base

```bash
mi_variable="solo en este shell"     # variable de shell, NO heredada
export MI_VAR="heredada por hijos"    # variable de entorno, SI heredada

echo $PATH                            # lista de directorios donde se buscan ejecutables
which node                            # muestra desde que directorio de PATH se ejecuta "node"

env                                   # lista todas las variables de entorno actuales
printenv HOME                         # muestra el valor de una variable especifica
```

## Como se usa en un proyecto real

1. Se exportan variables de entorno necesarias para que aplicaciones hijas (como una API Node) las lean, en vez de pasarlas solo como variables de shell locales.
2. Se agrega un directorio nuevo a `PATH` (en `.bashrc` o equivalente) cuando se instala una herramienta manualmente que no queda accesible por defecto.
3. Se diagnostica el error "command not found" verificando primero si el directorio del ejecutable esta incluido en `PATH`.
4. Se documenta claramente que variables de entorno necesita un proyecto, conectando con lo visto en `ConfigModule` de NestJS y `.env` de Docker Compose.
5. Se distingue entre variables definidas en `.bashrc` (por sesion interactiva) y las inyectadas por el sistema de despliegue (systemd, Docker), evitando depender de configuracion de shell personal para produccion.

## Criterio junior

Un junior debe entender la diferencia entre una variable de shell simple y una exportada, y poder revisar el valor de `PATH` o cualquier otra variable de entorno.

## Criterio mid

Un mid debe diagnosticar un error "command not found" revisando `PATH`, y configurar variables de entorno persistentes en los archivos de configuracion del shell.

## Criterio senior

Un senior debe estandarizar como se inyectan variables de entorno en produccion (systemd, Docker, orquestador) evitando depender de configuracion de shell personal, y documentar claramente el contrato de configuracion de cada servicio.

## Errores comunes

- Definir una variable sin `export` y sorprenderse de que un proceso hijo (como un script llamado desde el shell) no puede verla.
- No revisar `PATH` ante un error "command not found", perdiendo tiempo reinstalando algo que en realidad ya estaba instalado pero no accesible.
- Depender de variables definidas solo en `.bashrc` personal para el funcionamiento de un servicio en produccion, rompiendo cuando corre bajo otro usuario o mecanismo (systemd, Docker).
- No documentar que variables de entorno necesita una aplicacion, dificultando el onboarding de nuevos integrantes del equipo.

## Practica

- [ ] Define una variable de shell simple y verifica que un script hijo no puede verla.
- [ ] Exporta esa misma variable y confirma que ahora si es heredada.
- [ ] Diagnostica un `PATH` incompleto agregando un directorio nuevo y verificando el efecto.
- [ ] Revisa donde se definen las variables de entorno persistentes en tu shell (`.bashrc` o equivalente).
- [ ] Documenta las variables de entorno que necesita un proyecto real tuyo.

## Preguntas de repaso

- Que diferencia hay entre una variable de shell y una variable de entorno exportada?
- Que hace la variable `PATH` y como se relaciona con el error "command not found"?
- Donde se definen variables de entorno persistentes entre sesiones de shell?
- Por que produccion no deberia depender de variables definidas en `.bashrc` personal de un usuario?
