---
tags:
  - Linux
  - Paquetes
  - Linux-Paquetes
Creado: 2026-07-28
Relacionado:
  - Gestion de Paquetes y Software
  - LINUX
---

# 04 - Gestores de Paquetes de Lenguaje en Linux

## Que vas a aprender

En este capitulo vas a entender como se relacionan los gestores de paquetes del **sistema operativo** (APT/DNF) con los gestores de paquetes de **lenguaje** (npm para Node, pip para Python, NuGet para .NET), y las buenas practicas para no mezclarlos incorrectamente.

## Explicacion

APT/DNF gestionan software a nivel de **sistema operativo**: librerias del sistema, herramientas de linea de comandos, servicios. npm (ya visto en profundidad en el libro de NestJS), pip, y NuGet gestionan dependencias a nivel de **proyecto/aplicacion**, generalmente instaladas en un directorio local del proyecto (`node_modules/`) o un entorno aislado, no mezcladas con paquetes del sistema.

Un error comun es instalar herramientas de Python con `apt install python3-algo` cuando en realidad se necesitaba una version especifica gestionada por `pip` dentro de un entorno virtual (`venv`), o instalar paquetes npm globalmente con `sudo npm install -g` cuando eso puede generar problemas de permisos y conflictos entre proyectos que necesitan versiones distintas de la misma herramienta. La practica recomendada es mantener los gestores de lenguaje aislados por proyecto (entornos virtuales de Python, `node_modules` local sin `-g` salvo excepciones justificadas).

## Modelo mental

Pensa el gestor de paquetes del sistema operativo como el suministro electrico general de un edificio (infraestructura compartida por todos los departamentos), y los gestores de paquetes de lenguaje como los electrodomesticos especificos que cada departamento (proyecto) instala dentro de su propio espacio, sin necesitar coordinar con el resto del edificio ni afectarlo.

## Ejemplo base

```bash
# Sistema: instala Python y Node como runtimes, gestionados por APT
sudo apt install python3 python3-venv nodejs npm

# Proyecto Python: entorno virtual aislado, dependencias NO globales
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Proyecto Node: dependencias locales al proyecto, sin necesitar sudo
npm install     # instala en ./node_modules, no globalmente
```

## Como se usa en un proyecto real

1. Se usa APT/DNF unicamente para instalar el **runtime** del lenguaje (Node, Python, .NET SDK) a nivel de sistema, nunca para gestionar dependencias especificas de un proyecto.
2. Se usan entornos virtuales de Python (`venv`) para aislar dependencias entre proyectos distintos que pueden necesitar versiones incompatibles de la misma libreria.
3. Se evita `sudo npm install -g` salvo para herramientas de CLI genuinamente globales (como el propio Nest CLI), prefiriendo dependencias locales de proyecto siempre que sea posible.
4. Se documenta claramente en el README de cada proyecto que gestor de lenguaje usa y como instalar sus dependencias, distinto de las dependencias de sistema necesarias.
5. Se conecta esto con lo ya visto en el capitulo de gestion de dependencias del libro de NestJS: el mismo principio de lockfiles y reproducibilidad aplica igual a `requirements.txt` de Python o `packages.lock.json` de .NET.

## Criterio junior

Un junior debe distinguir cuando instalar algo con APT/DNF (nivel sistema) versus con el gestor del lenguaje especifico (nivel proyecto).

## Criterio mid

Un mid debe usar entornos virtuales de Python correctamente, y evitar instalaciones globales innecesarias de paquetes npm que puedan generar conflictos entre proyectos.

## Criterio senior

Un senior debe estandarizar la separacion clara entre dependencias de sistema y de proyecto en toda la infraestructura del equipo, documentando el proceso de setup de cada tipo de proyecto de forma reproducible.

## Errores comunes

- Instalar dependencias de un proyecto especifico con APT/DNF en vez del gestor de lenguaje correspondiente, mezclando responsabilidades que deberian estar separadas.
- No usar entornos virtuales de Python, generando conflictos de versiones entre distintos proyectos que corren en el mismo sistema.
- Abusar de `sudo npm install -g` para dependencias que en realidad deberian ser locales al proyecto.
- No documentar claramente la diferencia entre dependencias de sistema y de proyecto en el README, confundiendo a nuevos integrantes del equipo.

## Practica

- [ ] Instala el runtime de un lenguaje (Node o Python) con APT/DNF a nivel de sistema.
- [ ] Crea un entorno virtual de Python para un proyecto y verifica que sus dependencias quedan aisladas.
- [ ] Instala dependencias de un proyecto Node localmente (sin `-g`) y verifica que quedan en `node_modules`.
- [ ] Identifica en un proyecto real la diferencia entre lo que necesita instalarse a nivel sistema y a nivel proyecto.
- [ ] Documenta el proceso de setup de un proyecto distinguiendo ambos niveles claramente.

## Preguntas de repaso

- Que diferencia de alcance hay entre APT/DNF y npm/pip/NuGet?
- Por que usar un entorno virtual de Python evita conflictos entre proyectos distintos?
- Que riesgo tiene abusar de instalaciones globales (`sudo npm install -g`) para dependencias de proyecto?
- Como documentarias la diferencia entre dependencias de sistema y de proyecto para un nuevo integrante del equipo?
