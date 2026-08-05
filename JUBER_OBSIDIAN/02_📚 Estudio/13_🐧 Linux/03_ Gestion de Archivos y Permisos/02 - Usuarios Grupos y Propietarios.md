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

# 02 - Usuarios Grupos y Propietarios

## Que vas a aprender

En este capitulo vas a aprender a gestionar **usuarios y grupos** del sistema: creacion, modificacion, pertenencia a grupos, y los archivos clave donde Linux almacena esta informacion (`/etc/passwd`, `/etc/group`, `/etc/shadow`).

## Explicacion

Cada usuario en Linux tiene un **UID** (User ID) numerico unico, y pertenece a un **grupo primario** ademas de opcionalmente varios **grupos secundarios**. `/etc/passwd` almacena informacion basica de cada usuario (nombre, UID, GID primario, home, shell), `/etc/group` define los grupos y sus miembros, y `/etc/shadow` (accesible solo por root) guarda las contraseñas cifradas, separadas de `/etc/passwd` por razones de seguridad (historicamente `/etc/passwd` era legible por todos).

Un usuario especial es **root** (UID 0), con permisos totales sobre el sistema, sin restriccion de permisos rwx. Por eso, siguiendo el principio de menor privilegio ya visto en el libro de Docker (usuario no-root dentro de containers), la practica estandar es trabajar con un usuario normal y usar `sudo` para elevar privilegios solo cuando es estrictamente necesario, en vez de operar como root todo el tiempo.

## Modelo mental

Pensa cada usuario como un empleado con una credencial de identificacion unica (UID), que pertenece a un departamento principal (grupo primario) y opcionalmente a comites adicionales (grupos secundarios) que le dan acceso a recursos compartidos especificos, sin necesitar ser transferido permanentemente a otro departamento.

## Ejemplo base

```bash
sudo useradd -m -s /bin/bash nuevo_usuario   # crea usuario con home y shell bash
sudo passwd nuevo_usuario                     # establece su contraseña

sudo usermod -aG docker nuevo_usuario         # agrega al grupo "docker" (secundario)
groups nuevo_usuario                          # muestra a que grupos pertenece

id nuevo_usuario                              # UID, GID y grupos, todo junto
cat /etc/passwd | grep nuevo_usuario          # entrada cruda en /etc/passwd
```

## Como se usa en un proyecto real

1. Se crea un usuario dedicado (no root) para cada servicio importante que corre en el servidor, aplicando el mismo principio de menor privilegio visto para containers.
2. Se usan grupos para dar acceso compartido a recursos especificos (por ejemplo, el grupo `docker` da permiso de usar Docker sin necesitar sudo en cada comando) sin dar acceso total de root.
3. Se audita periodicamente quien pertenece a grupos privilegiados (`sudo`, `docker`), removiendo accesos que ya no son necesarios.
4. Se nunca comparte una cuenta de usuario entre varias personas, manteniendo trazabilidad clara de quien hizo que en el sistema.
5. Se documenta la convencion de usuarios y grupos del equipo (que usuario corre cada servicio, que grupos existen y para que) como parte de la documentacion de infraestructura.

## Criterio junior

Un junior debe poder crear un usuario nuevo, agregarlo a un grupo, y verificar su pertenencia con `groups` o `id`.

## Criterio mid

Un mid debe diseñar la estructura de usuarios y grupos de un servidor nuevo, asignando el usuario correcto a cada servicio segun el principio de menor privilegio.

## Criterio senior

Un senior debe auditar y mantener la higiene de usuarios y grupos de toda la infraestructura, revocando accesos innecesarios y estableciendo el proceso de onboarding/offboarding de cuentas de sistema.

## Errores comunes

- Correr servicios como root sin necesidad, cuando podrian correr con un usuario dedicado de menor privilegio.
- Compartir una unica cuenta de usuario entre varias personas del equipo, perdiendo trazabilidad de acciones individuales.
- No auditar periodicamente la pertenencia a grupos privilegiados, dejando accesos innecesarios acumulados con el tiempo.
- No documentar la convencion de usuarios y grupos del sistema, dificultando el entendimiento para nuevos administradores.

## Practica

- [ ] Crea un usuario nuevo con home y shell, y establece su contraseña.
- [ ] Agrega ese usuario a un grupo secundario y verifica su pertenencia.
- [ ] Revisa `/etc/passwd` y `/etc/group` para entender su formato real.
- [ ] Audita que usuarios de un sistema real o de practica pertenecen a grupos privilegiados.
- [ ] Documenta la convencion de usuarios que aplicarias para servicios de tu infraestructura.

## Preguntas de repaso

- Que informacion almacena `/etc/passwd` y por que las contraseñas cifradas viven en `/etc/shadow` por separado?
- Que diferencia hay entre el grupo primario y los grupos secundarios de un usuario?
- Por que es mejor practica correr servicios con un usuario dedicado en vez de como root?
- Como auditarias que usuarios tienen acceso a grupos privilegiados en un sistema?
