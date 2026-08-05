---
tags:
  - Linux
  - Bash
  - Linux-Texto
Creado: 2026-07-28
Relacionado:
  - Editores y Manipulacion de Texto
  - LINUX
---

# 01 - Vim Fundamentos

## Que vas a aprender

En este capitulo vas a aprender lo esencial de **Vim**, el editor de texto disponible practicamente en cualquier servidor Linux (a diferencia de editores graficos), incluyendo sus **modos** de operacion y los comandos minimos para sobrevivir editando un archivo de configuracion remoto.

## Explicacion

Vim tiene varios **modos**: **modo normal** (el modo por defecto al abrir, donde las teclas son comandos de navegacion/edicion, no texto), **modo insercion** (`i` para entrar, donde escribis texto normalmente), y **modo comando** (`:`, para acciones como guardar o salir). Esta separacion de modos es lo que hace a Vim inicialmente confuso para quien viene de editores graficos, pero tambien lo que permite editar texto muy eficientemente una vez que se domina.

Los comandos minimos indispensables para sobrevivir son: `i` para insertar texto, `Esc` para volver a modo normal, `:wq` para guardar y salir, `:q!` para salir sin guardar cambios, y `dd` para borrar una linea completa. Con solo estos comandos ya se puede editar cualquier archivo de configuracion en un servidor remoto sin quedar atrapado sin saber como salir (un problema clasico de quien nunca uso Vim).

## Modelo mental

Pensa Vim como un instrumento musical con distintos "estados de la mano": en modo normal tus dedos estan listos para tocar acordes de navegacion y edicion rapida (cada tecla es un comando), en modo insercion tus dedos simplemente escriben notas una tras otra (texto normal). Cambiar de modo con `Esc` es como sacar la mano del instrumento antes de tocar el siguiente acorde.

## Ejemplo base

```bash
vim archivo.conf

# En modo normal:
i          # entra a modo insercion (empezar a escribir texto)
Esc        # vuelve a modo normal

dd         # borra la linea actual
yy         # copia (yank) la linea actual
p          # pega despues de la linea actual
u          # deshacer

:wq        # guarda y sale
:q!        # sale SIN guardar cambios
/palabra   # busca "palabra" hacia adelante (n para siguiente coincidencia)
```

## Como se usa en un proyecto real

1. Se usa Vim para ediciones rapidas de configuracion en servidores remotos donde no hay entorno grafico ni editor sincronizado (via SSH directo).
2. Se memorizan los comandos minimos de supervivencia (`i`, `Esc`, `:wq`, `:q!`) como base indispensable, sin necesidad de dominar Vim completamente desde el principio.
3. Se usa `/palabra` para buscar rapidamente dentro de un archivo de configuracion largo sin necesitar scrollear manualmente.
4. Se evita entrar en panico si Vim se abre inesperadamente (por ejemplo, al ejecutar `git commit` sin editor configurado): se sabe salir con `:q!` sin dejar cambios accidentales.
5. Se profundiza en Vim mas alla de lo basico solo si se edita texto frecuentemente por terminal, dado que su verdadero poder aparece con practica sostenida.

## Criterio junior

Un junior debe poder abrir un archivo con Vim, entrar a modo insercion, escribir texto, y guardar/salir correctamente sin quedar atrapado.

## Criterio mid

Un mid debe usar comandos basicos de navegacion y edicion (`dd`, `yy`, `p`, `/busqueda`) para editar archivos de configuracion eficientemente sin depender de un editor grafico.

## Criterio senior

Un senior debe poder trabajar comodamente en cualquier servidor remoto sin depender de herramientas graficas, usando Vim como editor por defecto disponible universalmente en entornos de produccion.

## Errores comunes

- Quedar "atrapado" en Vim sin saber como salir, un problema clasico de quien nunca aprendio los comandos minimos.
- Intentar escribir texto directamente en modo normal, generando comandos no deseados en vez de texto.
- Guardar cambios accidentales con `:wq` cuando en realidad se queria descartarlos con `:q!`.
- No aprovechar busqueda (`/palabra`) en archivos largos, perdiendo tiempo scrolleando manualmente.

## Practica

- [ ] Abre un archivo de practica con Vim, edita texto, y guarda los cambios con `:wq`.
- [ ] Practica salir sin guardar con `:q!` despues de hacer cambios no deseados.
- [ ] Usa `dd`, `yy` y `p` para mover lineas dentro de un archivo.
- [ ] Busca una palabra especifica dentro de un archivo largo con `/palabra`.
- [ ] Anota los comandos minimos que memorizarias para "sobrevivir" en cualquier servidor con Vim.

## Preguntas de repaso

- Que diferencia hay entre modo normal y modo insercion en Vim?
- Como saldrias de Vim sin guardar cambios accidentales?
- Que hacen los comandos `dd`, `yy` y `p`?
- Por que Vim sigue siendo relevante pese a la existencia de editores graficos mas comodos?
