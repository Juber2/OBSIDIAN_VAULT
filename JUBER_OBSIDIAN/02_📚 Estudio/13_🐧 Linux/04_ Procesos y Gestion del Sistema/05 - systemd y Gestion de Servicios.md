---
tags:
  - Linux
  - Procesos
  - Linux-Procesos
Creado: 2026-07-28
Relacionado:
  - Procesos y Gestion del Sistema
  - LINUX
---

# 05 - systemd y Gestion de Servicios

## Que vas a aprender

En este capitulo vas a aprender a gestionar servicios de larga vida con **systemd**: `systemctl` para controlar servicios, y como crear una **unit file** propia para correr una aplicacion como servicio administrado del sistema.

## Explicacion

Ya vimos systemd como PID 1 en el capitulo de Fundamentos (boot process); aca profundizamos en su uso practico del dia a dia: `systemctl start/stop/restart/status` controla servicios individuales, `systemctl enable` los configura para arrancar automaticamente en cada boot, y `systemctl daemon-reload` recarga la configuracion despues de modificar una unit file.

Una **unit file** (`.service`) es un archivo de configuracion declarativo que le dice a systemd como correr una aplicacion: que comando ejecutar, con que usuario, que hacer si falla (reinicio automatico), y de que otros servicios depende. Esto reemplaza al patron manual de `nohup comando &` (visto en el tema anterior) para cualquier proceso que deba correr de forma confiable en produccion, con reinicio automatico ante fallos y arranque automatico en cada boot del servidor.

## Modelo mental

Pensa una unit file de systemd como el contrato de trabajo formal de un empleado permanente: define exactamente que trabajo hace, bajo que credenciales (usuario), que pasa si falta al trabajo sin avisar (politica de reinicio), y de que otros departamentos depende para poder empezar su turno. Esto es mucho mas confiable que simplemente decirle a alguien "andá y trabajá" de forma informal (`nohup &`).

## Ejemplo base

```ini
# /etc/systemd/system/mi-api.service
[Unit]
Description=Mi API Node.js
After=network.target postgresql.service

[Service]
Type=simple
User=api-user
WorkingDirectory=/opt/mi-api
ExecStart=/usr/bin/node dist/main.js
Restart=on-failure
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload         # recarga configuracion despues de crear/editar la unit
sudo systemctl enable mi-api          # arranca automaticamente en cada boot
sudo systemctl start mi-api           # inicia el servicio ahora
sudo systemctl status mi-api          # estado actual, incluyendo logs recientes
journalctl -u mi-api -f                # sigue los logs de este servicio en tiempo real
```

## Como se usa en un proyecto real

1. Se crea una unit file para cualquier aplicacion propia que deba correr de forma confiable en un servidor, en vez de usar `nohup` o screen/tmux para procesos de produccion.
2. Se configura `Restart=on-failure` para que systemd reinicie automaticamente el servicio si falla, similar a la `restart_policy` vista en Docker/Swarm.
3. Se define `User=` con un usuario dedicado no-root para el servicio, aplicando el mismo principio de menor privilegio visto en capitulos anteriores.
4. Se declaran dependencias con `After=`/`Requires=` cuando el servicio necesita que otro (como una base de datos local) ya este disponible.
5. Se usa `journalctl -u nombre-servicio` como primera herramienta de diagnostico ante problemas de un servicio administrado por systemd.

## Criterio junior

Un junior debe poder controlar un servicio existente con `systemctl start/stop/status`, y revisar sus logs con `journalctl -u`.

## Criterio mid

Un mid debe crear una unit file propia para una aplicacion, configurando usuario, politica de reinicio, y variables de entorno correctamente.

## Criterio senior

Un senior debe diseñar la configuracion de dependencias entre servicios propios en systemd, estableciendo el estandar de unit files del equipo (usuario no-root, politicas de reinicio, logging) para toda la infraestructura.

## Errores comunes

- Usar `nohup` o screen/tmux para correr una aplicacion de produccion en vez de configurarla como servicio systemd con reinicio automatico.
- No configurar `Restart=on-failure`, dejando que un servicio caido permanezca caido hasta que alguien lo note manualmente.
- Correr el servicio sin especificar `User=`, dejandolo correr como root sin necesidad.
- Olvidar `systemctl daemon-reload` despues de crear o modificar una unit file, causando que los cambios no se apliquen.

## Practica

- [ ] Crea una unit file propia para una aplicacion de practica (por ejemplo, una API Node simple).
- [ ] Configura `Restart=on-failure` y verifica que el servicio se reinicia automaticamente ante un fallo simulado.
- [ ] Habilita el servicio para arrancar automaticamente en boot con `systemctl enable`.
- [ ] Sigue los logs del servicio en tiempo real con `journalctl -u`.
- [ ] Documenta una plantilla de unit file estandar para tu equipo, con usuario no-root y politica de reinicio.

## Preguntas de repaso

- Que ventaja da administrar una aplicacion como servicio systemd sobre correrla con `nohup` en background?
- Que hace `Restart=on-failure` en una unit file, y como se relaciona con lo visto en Docker/Swarm?
- Por que es importante especificar un usuario no-root en la configuracion de un servicio?
- Que comando revisarias primero ante un problema con un servicio administrado por systemd?
