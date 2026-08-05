---
tags:
  - Linux
  - Redes
  - Linux-Redes
Creado: 2026-07-28
Relacionado:
  - Redes
  - LINUX
---

# 04 - SSH y Acceso Remoto Seguro

## Que vas a aprender

En este capitulo vas a dominar **SSH**, la herramienta fundamental de administracion remota de Linux: autenticacion por clave publica/privada, configuracion basica de `sshd`, y **port forwarding** para tuneles seguros.

## Explicacion

SSH (Secure Shell) da acceso remoto cifrado a un servidor Linux. La autenticacion por **contraseña** es funcional pero menos segura y menos comoda que la autenticacion por **par de claves**: se genera un par (`ssh-keygen`), la clave **privada** se queda siempre en tu maquina (nunca se comparte), y la clave **publica** se copia al servidor (`~/.ssh/authorized_keys`), permitiendo conectarse sin escribir contraseña cada vez, con seguridad criptografica mucho mayor.

El **port forwarding** de SSH permite crear tuneles seguros: forwarding local (`-L`) expone un servicio remoto en un puerto local (util para acceder a una base de datos que solo escucha en localhost del servidor remoto), forwarding remoto (`-R`) hace lo inverso. Esto conecta directamente con el capitulo de Redes del libro de Docker: es una forma de acceder de forma segura a servicios que, correctamente, no publican su puerto directamente al exterior.

## Modelo mental

Pensa la autenticacion por clave SSH como una cerradura de alta seguridad donde tu llave privada es la unica que fisicamente encaja: nadie puede copiar la cerradura (clave publica) y fabricar una llave que funcione sin tener acceso a la llave privada original. El port forwarding es como cavar un tunel seguro y privado directamente hasta una habitacion especifica de un edificio remoto, sin necesitar abrir esa habitacion al publico general.

## Ejemplo base

```bash
ssh-keygen -t ed25519 -C "mi-email@ejemplo.com"     # genera un par de claves moderno
ssh-copy-id usuario@servidor                          # copia la clave publica al servidor remoto

ssh usuario@servidor                                   # conecta usando la clave (sin contraseña)
ssh -i ~/.ssh/clave_especifica usuario@servidor        # usa una clave especifica no default

# Port forwarding local: accede a Postgres del servidor remoto (solo en localhost ahi) desde tu maquina
ssh -L 5432:localhost:5432 usuario@servidor
```

```
# ~/.ssh/config: simplifica conexiones frecuentes
Host miservidor
    HostName 203.0.113.10
    User juber
    IdentityFile ~/.ssh/clave_especifica
```

## Como se usa en un proyecto real

1. Se usa siempre autenticacion por clave en vez de contraseña para servidores de produccion, ademas de deshabilitar login por contraseña completamente en `sshd_config` (ver capitulo de Seguridad).
2. Se usa `~/.ssh/config` para simplificar conexiones frecuentes a multiples servidores, sin necesitar recordar direcciones IP o rutas de claves cada vez.
3. Se usa port forwarding local para acceder de forma segura a bases de datos u otros servicios que correctamente no publican su puerto al exterior del servidor.
4. Se genera un par de claves distinto por proposito o por maquina (no reusar la misma clave en todos lados), facilitando revocar acceso especifico si una clave se compromete.
5. Se conecta este conocimiento con las mejores practicas de acceso a servicios internos vistas en el capitulo de Redes de Docker, aplicando el mismo principio de "no exponer directamente lo que no necesita ser publico".

## Criterio junior

Un junior debe poder generar un par de claves SSH, copiarlas a un servidor, y conectarse sin necesitar contraseña.

## Criterio mid

Un mid debe configurar `~/.ssh/config` para simplificar conexiones frecuentes, y usar port forwarding local para acceder a servicios internos de forma segura.

## Criterio senior

Un senior debe diseñar la politica de gestion de claves SSH de todo el equipo (rotacion, revocacion, claves distintas por proposito) y deshabilitar autenticacion por contraseña en toda la infraestructura de produccion.

## Errores comunes

- Seguir usando autenticacion por contraseña en servidores de produccion en vez de migrar a claves, dejando el sistema mas expuesto a ataques de fuerza bruta.
- Reusar la misma clave SSH en todos los servidores y proyectos, dificultando revocar acceso especifico si una clave se compromete.
- No usar `~/.ssh/config`, retipeando direcciones IP y opciones de conexion cada vez en vez de simplificar el flujo diario.
- No aprovechar port forwarding, publicando puertos de servicios internos directamente al exterior cuando un tunel SSH seria mas seguro.

## Practica

- [ ] Genera un par de claves SSH moderno (`ed25519`) y copialo a un servidor de practica.
- [ ] Configura `~/.ssh/config` para simplificar la conexion a ese servidor.
- [ ] Usa port forwarding local para acceder a un servicio que solo escucha en localhost del servidor remoto.
- [ ] Investiga como deshabilitarias autenticacion por contraseña en `sshd_config` (sin aplicarlo aun, eso es del capitulo de Seguridad).
- [ ] Anota tu politica personal de gestion de claves SSH (una por maquina, por proposito, etc.).

## Preguntas de repaso

- Por que la autenticacion por clave es mas segura que por contraseña en SSH?
- Que archivo del servidor almacena las claves publicas autorizadas para un usuario?
- Que problema resuelve el port forwarding local de SSH en el contexto de servicios internos?
- Que ventaja da usar claves distintas por proposito o maquina en vez de reusar la misma en todos lados?
