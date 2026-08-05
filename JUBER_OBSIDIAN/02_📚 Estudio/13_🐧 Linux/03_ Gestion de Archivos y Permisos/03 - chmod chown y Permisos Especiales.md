---
tags:
  - Linux
  - Permisos
  - Linux-Permisos
Creado: 2026-07-28
Relacionado:
  - Gestion de Archivos y Permisos
  - LINUX
---

# 03 - chmod chown y Permisos Especiales

## Que vas a aprender

En este capitulo vas a dominar `chmod` y `chown` para modificar permisos y propietarios, tanto en notacion **simbolica** como **octal**, y los tres **permisos especiales**: setuid, setgid, y sticky bit.

## Explicacion

`chmod` cambia permisos de dos formas: **simbolica** (`chmod u+x archivo` agrega ejecucion al dueño, `chmod g-w` quita escritura al grupo) o **octal** (`chmod 755 archivo`, donde cada digito representa la suma de r=4, w=2, x=1 para dueño/grupo/otros respectivamente). `chown` cambia el dueño y/o grupo de un archivo (`chown usuario:grupo archivo`).

Los **permisos especiales** van mas alla del modelo rwx basico: **setuid** (en un ejecutable) hace que corra con los privilegios de su dueño en vez de quien lo ejecuta (usado por `passwd`, que necesita privilegios de root para modificar `/etc/shadow` aunque lo ejecute un usuario normal); **setgid** en un directorio hace que los archivos nuevos creados ahi hereden el grupo del directorio en vez del grupo del usuario que los crea, util para directorios compartidos por equipo; el **sticky bit** en un directorio (como `/tmp`) impide que un usuario borre archivos de otros usuarios aunque tenga permiso de escritura en el directorio.

## Modelo mental

Pensa setuid como un guardia de seguridad temporal que, al cruzar cierta puerta especifica, adquiere automaticamente la credencial del dueño del edificio solo para esa tarea puntual, sin importar quien sea realmente. El sticky bit es como una regla especial de un espacio compartido: "cualquiera puede dejar cosas aca, pero solo el dueño de cada cosa puede llevarsela de vuelta".

## Ejemplo base

```bash
chmod 755 script.sh          # rwxr-xr-x: dueño todo, grupo y otros solo leen/ejecutan
chmod u+x script.sh           # agrega ejecucion solo al dueño, sin tocar el resto
chown juber:developers archivo.txt   # cambia dueño y grupo

chmod 4755 /usr/bin/passwd    # setuid (el 4 inicial): corre con privilegios del dueño (root)
chmod 2775 /var/proyecto      # setgid en un directorio: archivos nuevos heredan el grupo
chmod 1777 /tmp                # sticky bit: cada usuario solo borra sus propios archivos
```

## Como se usa en un proyecto real

1. Se usa notacion octal para cambios completos de permisos (`chmod 644 archivo`) y simbolica para ajustes puntuales (`chmod g+w archivo`), segun cual sea mas clara en cada caso.
2. Se usa `chown` para corregir el propietario de archivos despues de una operacion de despliegue que los creo con un usuario incorrecto.
3. Se aplica setgid en directorios compartidos por un equipo, para que archivos nuevos hereden automaticamente el grupo correcto sin necesitar `chown` manual cada vez.
4. Se entiende el sticky bit de `/tmp` como la razon por la cual multiples usuarios pueden usar ese directorio compartido sin poder borrarse archivos entre si.
5. Se evita usar setuid en scripts propios sin entender completamente sus implicancias de seguridad, dado que un script con setuid mal escrito es un vector de escalada de privilegios.

## Criterio junior

Un junior debe poder usar `chmod` en notacion octal basica y `chown` para cambiar propietario y grupo de un archivo.

## Criterio mid

Un mid debe usar notacion simbolica para ajustes puntuales de permisos, y configurar setgid en directorios compartidos de equipo.

## Criterio senior

Un senior debe evaluar criticamente cualquier uso de setuid en scripts o binarios propios por su riesgo de seguridad, y diseñar esquemas de permisos con setgid y sticky bit apropiados para infraestructura compartida entre equipos.

## Errores comunes

- Usar `chmod 777` como solucion rapida ante un problema de permisos, otorgando permisos totales a todos sin necesidad real, un riesgo de seguridad serio.
- No entender la diferencia entre notacion octal y simbolica, generando cambios de permisos mas amplios o mas restrictivos de lo que se pretendia.
- Usar setuid en un script propio sin entender sus riesgos, creando una vulnerabilidad potencial de escalada de privilegios.
- No aplicar setgid en directorios compartidos de equipo, generando inconsistencia de grupo en archivos creados por distintos usuarios.

## Practica

- [ ] Cambia permisos de un archivo usando tanto notacion octal como simbolica y compara el resultado.
- [ ] Cambia el propietario y grupo de un archivo con `chown`.
- [ ] Configura setgid en un directorio de practica y verifica que archivos nuevos heredan el grupo correcto.
- [ ] Investiga por que `/tmp` tiene sticky bit activado por defecto en la mayoria de distribuciones.
- [ ] Identifica un caso donde `chmod 777` fue (o seria) una mala solucion, y proponé la alternativa correcta.

## Preguntas de repaso

- Como se calcula el valor octal de un permiso (por ejemplo, por que 755 significa rwxr-xr-x)?
- Que hace setuid en un ejecutable y por que representa un riesgo de seguridad si se usa mal?
- Que problema resuelve setgid en un directorio compartido por un equipo?
- Por que `chmod 777` es casi siempre una mala practica de seguridad?
