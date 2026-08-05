---
tags:
  - Linux
  - Proyectos
  - Linux-Proyectos
Creado: 2026-07-28
Relacionado:
  - Proyectos
  - LINUX
---

# 01 - Proyecto Configurar un Servidor Web Completo

## Que vas a construir

Un servidor Linux configurado desde cero para alojar una aplicacion web real, aplicando de punta a punta los capitulos de Fundamentos, Shell, Permisos, Procesos, Redes y Gestion de Paquetes.

## Alcance funcional

- Servidor con usuario no-root creado, con permisos de sudo apropiados.
- Nginx instalado y configurado como reverse proxy hacia una aplicacion backend (NestJS, .NET, o similar).
- Firewall basico configurado con politica de denegar por defecto.
- Acceso SSH configurado por clave, con el servicio administrado como unidad systemd.
- La aplicacion backend corriendo como servicio systemd propio, con reinicio automatico.

## Checklist tecnica

- [ ] Ningun proceso de aplicacion corre como root; cada uno tiene su usuario dedicado.
- [ ] El firewall permite unicamente los puertos estrictamente necesarios (SSH, HTTP/HTTPS).
- [ ] Nginx redirige correctamente hacia el backend, verificado con una request real.
- [ ] El servicio backend tiene `Restart=on-failure` configurado y probado (matando el proceso manualmente y verificando reinicio).
- [ ] Se puede conectar por SSH usando exclusivamente autenticacion por clave.

## Criterio de exito

El proyecto se considera completo cuando, desde una maquina limpia, se puede seguir un documento de setup propio (escrito durante el proyecto) para reproducir exactamente el mismo servidor, y la aplicacion responde correctamente a traves de Nginx con el backend administrado por systemd.

## Errores comunes a evitar

- Dejar el servidor con autenticacion SSH por contraseña habilitada.
- Correr el backend directamente con `node`/`dotnet` en una sesion de terminal en vez de como servicio systemd administrado.
- No verificar el firewall antes de aplicarlo, arriesgando quedar bloqueado del propio servidor.

## Extension opcional

Aplicar hardening basico de SSH (deshabilitar root login, fail2ban) como puente hacia el proyecto de Seguridad mas adelante en este capitulo.
