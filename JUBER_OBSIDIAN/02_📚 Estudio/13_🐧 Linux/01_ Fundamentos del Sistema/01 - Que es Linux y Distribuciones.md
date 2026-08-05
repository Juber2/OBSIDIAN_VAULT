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

# 01 - Que es Linux y Distribuciones

## Que vas a aprender

En este capitulo vas a entender **que es Linux** en realidad (un kernel, no un sistema operativo completo por si solo), y como las **distribuciones** empaquetan ese kernel junto con herramientas para formar un sistema operativo usable.

## Explicacion

Linux, estrictamente, es solo el **kernel**: el nucleo que gestiona hardware, procesos, memoria y el sistema de archivos a bajo nivel. Lo que la gente llama "Linux" en el dia a dia (Ubuntu, Debian, Fedora, Arch) es en realidad una **distribucion**: el kernel Linux combinado con un gestor de paquetes, utilidades GNU (bash, coreutils), y a menudo un entorno de escritorio, empaquetados por una organizacion o comunidad especifica.

Las distribuciones se agrupan en familias segun su gestor de paquetes y filosofia: la familia **Debian/Ubuntu** usa `apt` y `.deb`, favoreciendo estabilidad y un ciclo de releases predecible; la familia **RedHat/Fedora/CentOS** usa `dnf`/`yum` y `.rpm`, comun en entornos empresariales; **Arch** favorece un modelo rolling-release con paquetes siempre actualizados, orientado a usuarios que quieren maximo control. Para servidores y DevOps, Debian/Ubuntu y RedHat/derivados dominan ampliamente el panorama profesional.

## Modelo mental

Pensa el kernel Linux como el motor de un auto, y cada distribucion como un fabricante distinto que toma ese mismo motor y le construye una carroceria, tablero e interior propios (herramientas, gestor de paquetes, filosofia de actualizacion). El motor es el mismo por debajo, pero la experiencia de manejo (uso diario) varia segun el fabricante.

## Ejemplo base

```bash
cat /etc/os-release      # identifica la distribucion y version exacta
uname -r                 # version del kernel Linux corriendo
uname -a                 # informacion completa del kernel y arquitectura
```

## Como se usa en un proyecto real

1. Se elige una distribucion segun el contexto: Ubuntu/Debian para la mayoria de proyectos nuevos por su ecosistema y documentacion amplia, RHEL/derivados cuando la empresa ya tiene ese estandar o necesita soporte comercial formal.
2. Se estandariza la distribucion usada entre desarrollo, staging y produccion, para minimizar diferencias de comportamiento entre ambientes (el mismo principio de paridad visto en el libro de Docker).
3. Se documenta la version exacta de la distribucion y el kernel usados en servidores de produccion, relevante para diagnosticar bugs especificos de version.
4. Se evita mezclar comandos de gestion de paquetes de distintas familias (por ejemplo, intentar usar `apt` en un servidor RHEL) sin adaptar la guia al contexto real.
5. Se investiga el ciclo de vida de soporte (LTS) de la distribucion elegida, para planificar actualizaciones antes de que una version quede sin parches de seguridad.

## Criterio junior

Un junior debe poder identificar la distribucion y version de un sistema Linux con `cat /etc/os-release`, y entender la diferencia conceptual entre kernel y distribucion.

## Criterio mid

Un mid debe elegir la distribucion apropiada para un proyecto nuevo segun el contexto (ecosistema, soporte, estandar de la empresa) y adaptar comandos segun la familia de gestor de paquetes correspondiente.

## Criterio senior

Un senior debe estandarizar la distribucion usada en toda la infraestructura del equipo, planificar el ciclo de vida de actualizaciones (LTS) de forma proactiva, y justificar la eleccion de distribucion como una decision de infraestructura documentada.

## Errores comunes

- Confundir "Linux" con una distribucion especifica, sin entender que el kernel es compartido entre todas.
- Mezclar comandos de gestion de paquetes de distintas familias sin adaptarlos al sistema real que se esta administrando.
- No verificar el ciclo de vida de soporte de una distribucion antes de desplegarla en produccion, arriesgando quedar sin parches de seguridad.
- Usar distribuciones distintas entre desarrollo y produccion sin evaluar el riesgo de comportamientos diferentes entre ambientes.

## Practica

- [ ] Identifica la distribucion y version de un sistema Linux (real o en una maquina virtual) con `/etc/os-release`.
- [ ] Verifica la version del kernel con `uname -r`.
- [ ] Investiga el ciclo de vida de soporte (LTS) de esa distribucion especifica.
- [ ] Compara los gestores de paquetes de al menos dos familias distintas (Debian vs RedHat).
- [ ] Anota que distribucion elegirias para un servidor de produccion nuevo y por que.

## Preguntas de repaso

- Que diferencia hay, estrictamente, entre el kernel Linux y una distribucion?
- Que gestor de paquetes usa la familia Debian/Ubuntu, y cual usa la familia RedHat/Fedora?
- Por que es importante conocer el ciclo de vida de soporte (LTS) de una distribucion antes de usarla en produccion?
- Que riesgo trae usar distribuciones distintas entre desarrollo y produccion?
