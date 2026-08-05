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

# 01 - Bash y el Shell Interactivo

## Que vas a aprender

En este capitulo vas a entender que es un **shell**, por que **Bash** es el mas usado en Linux, y como funciona el ciclo basico de interaccion: leer un comando, interpretarlo, ejecutarlo, y mostrar el resultado.

## Explicacion

Un shell es el programa que interpreta los comandos que escribis y se los pasa al kernel para ejecutarlos, actuando como intermediario entre el usuario y el sistema. **Bash** (Bourne Again Shell) es el shell por defecto en la mayoria de distribuciones Linux, sucesor del shell original de Unix, con features de scripting, historial, y expansion de nombres de archivo. Otros shells existen (`zsh`, `fish`) con mejoras de usabilidad, pero Bash sigue siendo el estandar universal, especialmente relevante porque los scripts de sistema y de despliegue casi siempre asumen Bash disponible.

El shell corre en un **loop**: muestra un prompt, lee lo que escribis, lo interpreta (identificando el comando, sus argumentos, y cualquier caracter especial como pipes o redirecciones), lo ejecuta creando un proceso hijo, y espera su resultado antes de volver a mostrar el prompt. Entender este ciclo basico ayuda a predecir el comportamiento de comandos complejos.

## Modelo mental

Pensa el shell como un traductor personal que esta parado entre vos y el sistema operativo: le decis lo que queres en un idioma comodo para vos (comandos y su sintaxis), y el traductor se encarga de comunicarselo correctamente al sistema, y de traerte de vuelta la respuesta.

## Ejemplo base

```bash
echo $SHELL          # que shell estas usando actualmente
bash --version        # version de Bash instalada

# El shell interpreta esto: identifica "echo" como comando, "Hola Mundo" como argumento
echo "Hola Mundo"
```

## Como se usa en un proyecto real

1. Se estandariza Bash como shell de scripts de despliegue y automatizacion, dado que es el mas ampliamente disponible en cualquier servidor Linux sin instalacion adicional.
2. Se personaliza el shell interactivo (con `zsh` u otro, si se prefiere) para el uso diario del desarrollador, sin que eso afecte los scripts que corren en servidores (que siguen usando `#!/bin/bash` explicito).
3. Se verifica la version de Bash disponible en un servidor antes de usar features modernas de scripting que puedan no estar soportadas en versiones muy antiguas.
4. Se entiende el shebang (`#!/bin/bash`) al inicio de un script como la forma de asegurar que se interprete con el shell correcto, sin importar cual sea el shell interactivo del usuario que lo ejecuta.
5. Se documenta que shell asume cada script del proyecto, evitando ambiguedad quando el equipo usa distintos shells interactivos en sus propias maquinas.

## Criterio junior

Un junior debe poder identificar que shell esta usando y ejecutar comandos basicos entendiendo el ciclo de lectura-interpretacion-ejecucion.

## Criterio mid

Un mid debe distinguir cuando un comportamiento inesperado se debe a diferencias entre shells (Bash vs zsh vs sh) y escribir scripts con el shebang correcto para asegurar compatibilidad.

## Criterio senior

Un senior debe estandarizar el shell usado en scripts de infraestructura de todo el equipo, documentando esa decision, y anticipar problemas de compatibilidad entre versiones de Bash en distintos servidores.

## Errores comunes

- Escribir un script asumiendo features de Bash sin el shebang correcto, fallando cuando se ejecuta con un shell distinto (`sh` en vez de `bash`).
- Confundir comportamientos especificos de un shell personalizado (zsh) con comportamiento estandar de Bash al escribir documentacion o scripts.
- No verificar la version de Bash disponible en un servidor antes de usar sintaxis moderna que puede no estar soportada.
- No entender el ciclo basico de interpretacion del shell, dificultando predecir como se comportara un comando con caracteres especiales.

## Practica

- [ ] Identifica tu shell actual y su version.
- [ ] Escribe un script simple con el shebang `#!/bin/bash` explicito.
- [ ] Ejecuta ese mismo script forzando `sh` en vez de `bash` y observa si hay diferencias de comportamiento.
- [ ] Investiga que diferencias practicas tiene `zsh` respecto a Bash para uso interactivo.
- [ ] Documenta que shell asumis en los scripts de tu proyecto actual.

## Preguntas de repaso

- Que rol cumple el shell entre el usuario y el kernel?
- Por que Bash sigue siendo el estandar para scripts de sistema pese a la existencia de shells mas modernos?
- Para que sirve el shebang `#!/bin/bash` al inicio de un script?
- Que pasos sigue el shell en su ciclo basico de interaccion con cada comando ingresado?
