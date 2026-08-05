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

# 02 - Sed y Expresiones Regulares

## Que vas a aprender

En este capitulo vas a aprender **sed** (stream editor), la herramienta estandar para transformar texto de forma no interactiva desde la linea de comandos o scripts, junto con las **expresiones regulares** basicas que lo hacen poderoso.

## Explicacion

`sed` procesa texto **linea por linea**, aplicando comandos de edicion sin necesitar abrir un editor interactivo, ideal para transformaciones automatizables dentro de scripts o pipelines. El uso mas comun es **buscar y reemplazar** (`s/patron/reemplazo/`), donde el patron puede ser una expresion regular: `.` coincide con cualquier caracter, `*` repite el elemento anterior cero o mas veces, `^` y `$` anclan al inicio/fin de linea, y grupos entre parentesis `\(...\)` permiten capturar y reusar partes del texto coincidente.

Un detalle critico de seguridad: `sed -i` modifica el archivo **in-place** (directamente, sin confirmacion), por lo que conviene probar primero sin `-i` (viendo el resultado en pantalla) o usar `sed -i.bak` para crear un respaldo automatico antes de aplicar el cambio real.

## Modelo mental

Pensa `sed` como una maquina de corregir textos en una linea de produccion: cada linea de texto pasa por la maquina, que aplica automaticamente una regla de correccion (buscar-y-reemplazar) sin que nadie tenga que leerla y corregirla manualmente una por una, procesando miles de lineas en el tiempo que tomaria corregir una sola a mano.

## Ejemplo base

```bash
sed 's/localhost/produccion.com/' config.txt
# muestra el resultado en pantalla, SIN modificar el archivo original

sed -i.bak 's/DEBUG=true/DEBUG=false/' .env
# modifica .env in-place, creando .env.bak como respaldo automatico

sed -n '5,10p' archivo.log        # imprime solo las lineas 5 a 10
sed '/^#/d' config.txt              # elimina lineas que empiezan con # (comentarios)

echo "usuario@dominio.com" | sed 's/\(.*\)@\(.*\)/Usuario: \1, Dominio: \2/'
# usa grupos capturados (\1, \2) para reordenar el texto
```

## Como se usa en un proyecto real

1. Se usa `sed` para transformaciones de configuracion automatizadas en scripts de despliegue (por ejemplo, reemplazar un placeholder por un valor real antes de copiar un archivo).
2. Se prueba siempre sin `-i` primero para verificar el resultado antes de aplicar el cambio real de forma irreversible.
3. Se usa `sed -i.bak` en operaciones criticas para tener un respaldo automatico inmediato ante un reemplazo incorrecto.
4. Se combinan expresiones regulares con grupos capturados para transformaciones mas complejas que un simple buscar-y-reemplazar literal.
5. Se documenta el comando `sed` usado en scripts compartidos, dado que la sintaxis de expresiones regulares puede ser dificil de leer para quien no esta familiarizado.

## Criterio junior

Un junior debe poder usar `sed` para un buscar-y-reemplazar simple, y entender la diferencia entre probar sin `-i` y aplicar con `-i`.

## Criterio mid

Un mid debe usar expresiones regulares con anclas (`^`, `$`) y comodines (`.`, `*`) para transformaciones mas complejas, y usar `-i.bak` para respaldos automaticos.

## Criterio senior

Un senior debe integrar `sed` en scripts de automatizacion de despliegue de forma segura, con verificacion previa y respaldo, documentando transformaciones complejas para que el equipo pueda mantenerlas.

## Errores comunes

- Usar `sed -i` directamente sin probar antes, arriesgando modificar un archivo de forma incorrecta e irreversible.
- No entender que `sed` procesa linea por linea, esperando que un patron multi-linea funcione sin configuracion adicional.
- Escribir expresiones regulares demasiado genericas que coinciden con mas texto del esperado.
- No usar `-i.bak` en operaciones criticas, perdiendo la posibilidad de revertir un cambio incorrecto rapidamente.

## Practica

- [ ] Usa `sed` para hacer un buscar-y-reemplazar simple, primero sin `-i` para verificar el resultado.
- [ ] Aplica el cambio con `sed -i.bak` y verifica que el respaldo se creo correctamente.
- [ ] Usa `sed -n` para imprimir solo un rango especifico de lineas de un archivo.
- [ ] Practica un reemplazo usando grupos capturados con expresiones regulares.
- [ ] Anota un caso real de tu trabajo donde `sed` automatizaria una tarea de configuracion repetitiva.

## Preguntas de repaso

- Por que es buena practica probar un comando `sed` sin `-i` antes de aplicarlo con `-i`?
- Que hace `sed -i.bak` de forma distinta a `sed -i` simple?
- Que representan los caracteres especiales `^`, `$`, `.` y `*` en una expresion regular basica?
- Como usarias grupos capturados para reordenar partes de un texto con `sed`?
