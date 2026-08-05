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

# 02 - Navegacion y Manejo de Archivos

## Que vas a aprender

En este capitulo vas a dominar los comandos esenciales de navegacion y manipulacion de archivos: `cd`, `ls`, `cp`, `mv`, `rm`, `mkdir`, y las convenciones de rutas absolutas vs relativas.

## Explicacion

Una **ruta absoluta** empieza desde la raiz (`/home/juber/proyecto`) y siempre apunta al mismo lugar sin importar donde estes parado; una **ruta relativa** (`proyecto/src`, `../otro-directorio`) depende del directorio actual desde donde se ejecuta el comando. Los atajos `.` (directorio actual) y `..` (directorio padre) son fundamentales para navegar eficientemente sin escribir rutas absolutas completas todo el tiempo.

Comandos como `rm` no tienen papelera de reciclaje por defecto: un archivo eliminado con `rm` desaparece inmediatamente y no es recuperable de forma simple, a diferencia de arrastrar algo a la papelera en una interfaz grafica. Esto hace que `rm -rf` (recursivo y forzado) sea uno de los comandos mas peligrosos del dia a dia si se usa con una ruta incorrecta.

## Modelo mental

Pensa las rutas absolutas como una direccion postal completa (siempre lleva al mismo lugar sin importar desde donde envies la carta), y las rutas relativas como decir "la casa de al lado" o "dos cuadras atras" (depende completamente de donde estes parado en ese momento).

## Ejemplo base

```bash
pwd                       # muestra el directorio actual (ruta absoluta)
cd /var/log                # navega a una ruta absoluta
cd ../otro-proyecto        # navega a una ruta relativa (sube un nivel, entra a otro)
cd -                       # vuelve al directorio anterior

ls -la                     # lista incluyendo ocultos, formato detallado
cp archivo.txt backup.txt  # copia un archivo
mv viejo.txt nuevo.txt     # renombra o mueve un archivo
mkdir -p carpeta/subcarpeta  # crea carpetas anidadas de una vez
rm -rf carpeta_vieja        # elimina recursivamente, SIN confirmacion ni papelera
```

## Como se usa en un proyecto real

1. Se usan rutas absolutas en scripts que deben funcionar sin importar desde donde se ejecuten, y rutas relativas en el uso interactivo diario por comodidad.
2. Se verifica siempre la ruta exacta (con `pwd` o revisando el comando completo) antes de ejecutar `rm -rf`, dado que no hay forma de deshacer ese comando.
3. Se usa `ls -la` habitualmente para ver archivos ocultos (que empiezan con `.`, como `.env` o `.gitignore`) que `ls` simple no muestra.
4. Se prefiere `mkdir -p` para crear estructuras de carpetas anidadas en un unico comando, sin necesitar crear cada nivel por separado.
5. Se documenta, en scripts compartidos con el equipo, si las rutas usadas son absolutas o relativas y desde donde se espera que el script se ejecute.

## Criterio junior

Un junior debe poder navegar el sistema de archivos con `cd`, listar contenido con `ls`, y realizar operaciones basicas de copia, movimiento y eliminacion de archivos.

## Criterio mid

Un mid debe distinguir cuando usar rutas absolutas vs relativas segun el contexto (script vs uso interactivo), y verificar comandos destructivos antes de ejecutarlos.

## Criterio senior

Un senior debe establecer convenciones de rutas en scripts compartidos del equipo, y disenar practicas de seguridad (como confirmaciones o dry-runs) para operaciones destructivas en scripts de automatizacion.

## Errores comunes

- Ejecutar `rm -rf` con una ruta incorrecta (por ejemplo, un espacio de mas cambiando el significado del comando) sin verificar antes, causando perdida de datos irreversible.
- Confundir rutas relativas y absolutas dentro de un script, causando que funcione solo si se ejecuta desde un directorio especifico sin documentarlo.
- No usar `ls -la` y no darse cuenta de archivos ocultos importantes (`.env`, `.git`) presentes en un directorio.
- Usar `mkdir` sin `-p` para estructuras anidadas y encontrarse con errores porque los directorios intermedios no existen.

## Practica

- [ ] Practica navegar usando rutas absolutas y relativas, incluyendo `cd -` para volver al directorio anterior.
- [ ] Lista un directorio con `ls -la` e identifica archivos ocultos.
- [ ] Crea una estructura de carpetas anidadas con `mkdir -p` en un unico comando.
- [ ] Verifica la ruta exacta con `pwd` antes de simular un `rm -rf` (usa un directorio de prueba descartable).
- [ ] Anota una regla propia de seguridad antes de ejecutar cualquier comando destructivo.

## Preguntas de repaso

- Que diferencia hay entre una ruta absoluta y una relativa?
- Por que `rm -rf` es uno de los comandos mas peligrosos del uso diario de Linux?
- Que muestra `ls -la` que `ls` simple no muestra?
- Que verificacion harias siempre antes de ejecutar un comando destructivo con una ruta?
