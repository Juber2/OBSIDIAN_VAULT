---
tags:
  - Linux
  - Bash
  - Linux-Scripting
Creado: 2026-07-28
Relacionado:
  - Scripting con Bash
  - LINUX
---

# 01 - Fundamentos de Bash Scripting

## Que vas a aprender

En este capitulo vas a aprender a escribir tus primeros **scripts de Bash** reales: estructura basica, el shebang, variables, y como hacerlos ejecutables, conectando todo lo visto en capitulos anteriores en algo reutilizable.

## Explicacion

Un script Bash es simplemente un archivo de texto con comandos, uno por linea (o separados por `;`), que empieza con el **shebang** `#!/bin/bash` (ya visto en el capitulo de Shell) para asegurar que se interprete con Bash sin importar cual sea el shell interactivo de quien lo ejecuta. Para poder ejecutarlo directamente (`./script.sh`) necesita permiso de ejecucion (`chmod +x script.sh`, ver capitulo de Permisos); alternativamente, se puede ejecutar sin ese permiso invocando el interprete explicitamente (`bash script.sh`).

Las variables en Bash se declaran sin tipo (`nombre="valor"`, sin espacios alrededor del `=`) y se usan con `$nombre` o `${nombre}` (esta ultima forma es mas segura cuando la variable esta junto a otro texto sin espacio, evitando ambiguedad). Los comentarios empiezan con `#`.

## Modelo mental

Pensa un script Bash como una receta de cocina escrita: cada linea es un paso que se ejecuta en orden exacto, de arriba hacia abajo, tal como lo harias manualmente en la terminal, pero guardado para no tener que repetir los mismos pasos escribiendolos de nuevo cada vez.

## Ejemplo base

```bash
#!/bin/bash
# Script simple de saludo con variable

nombre="Mundo"
echo "Hola, ${nombre}!"

fecha_actual=$(date +%Y-%m-%d)   # captura la salida de un comando en una variable
echo "Hoy es: $fecha_actual"
```

```bash
chmod +x saludo.sh    # otorga permiso de ejecucion
./saludo.sh            # ejecuta el script
```

## Como se usa en un proyecto real

1. Se usa `#!/bin/bash` como primera linea de todo script, asegurando comportamiento consistente sin importar el shell interactivo de quien lo ejecute.
2. Se otorga permiso de ejecucion (`chmod +x`) a scripts que se ejecutaran directamente, versus invocarlos siempre con `bash script.sh`.
3. Se usa `$(comando)` (command substitution) para capturar la salida de un comando en una variable, en vez de la sintaxis antigua con backticks.
4. Se prefiere `${variable}` sobre `$variable` cuando hay ambiguedad sobre donde termina el nombre de la variable dentro de un string mas grande.
5. Se comentan los scripts con `#` explicando el proposito general al inicio, especialmente si se comparten con el equipo.

## Criterio junior

Un junior debe poder escribir un script Bash simple con shebang, variables, y hacerlo ejecutable con `chmod +x`.

## Criterio mid

Un mid debe usar command substitution para capturar salidas de comandos en variables, y estructurar scripts con comentarios claros para el equipo.

## Criterio senior

Un senior debe establecer convenciones de scripting para el equipo (shebang estandar, estilo de comentarios, estructura de scripts) que faciliten mantenimiento colaborativo a largo plazo.

## Errores comunes

- Olvidar el shebang, causando que el script se ejecute con un shell distinto al esperado si se invoca de cierta forma.
- Dejar espacios alrededor del `=` al declarar una variable (`nombre = "valor"`), lo cual en Bash es un error de sintaxis, no una asignacion valida.
- No otorgar permiso de ejecucion al script antes de intentar correrlo directamente con `./script.sh`.
- Usar la sintaxis antigua de backticks para command substitution en vez de `$(...)`, mas legible y anidable.

## Practica

- [ ] Escribe un script Bash simple con shebang, una variable, y un `echo` que la use.
- [ ] Otorga permiso de ejecucion al script y corrélo directamente.
- [ ] Captura la salida de un comando (como `date` o `whoami`) en una variable con command substitution.
- [ ] Agrega comentarios explicando el proposito del script.
- [ ] Anota una convencion de estilo que aplicarias a todos los scripts de tu equipo.

## Preguntas de repaso

- Que hace el shebang `#!/bin/bash` y por que es importante incluirlo?
- Por que `nombre = "valor"` (con espacios) es un error de sintaxis en Bash?
- Como capturarias la salida de un comando en una variable?
- Que diferencia hay entre `$variable` y `${variable}`, y cuando conviene usar la segunda forma?
