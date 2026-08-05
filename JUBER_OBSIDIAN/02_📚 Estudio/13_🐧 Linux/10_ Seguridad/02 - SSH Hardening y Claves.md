---
tags:
  - Linux
  - Seguridad
  - Linux-Seguridad
Creado: 2026-07-28
Relacionado:
  - Seguridad
  - LINUX
---

# 02 - SSH Hardening y Claves

## Que vas a aprender

En este capitulo vas a profundizar en el **hardening de SSH**, el servicio mas atacado en cualquier servidor con IP publica: deshabilitar login por contraseña y root, cambiar el puerto por defecto, y otras configuraciones criticas de `sshd_config`.

## Explicacion

El servicio SSH escuchando en el puerto 22 con autenticacion por contraseña habilitada es el objetivo mas comun de ataques de fuerza bruta automatizados en internet. El hardening basico e indispensable incluye: `PasswordAuthentication no` (forzando exclusivamente autenticacion por clave, ya vista en el capitulo de Redes), `PermitRootLogin no` (nadie deberia poder conectarse directamente como root; se conecta con un usuario normal y usa sudo, aplicando lo visto en el tema anterior), y opcionalmente cambiar el puerto por defecto (reduce el ruido de escaneos automatizados masivos, aunque no es una medida de seguridad real por si sola, solo reduccion de ruido).

Otras configuraciones relevantes: `AllowUsers`/`AllowGroups` restringe explicitamente que usuarios pueden conectarse por SSH, `MaxAuthTries` limita intentos fallidos antes de cortar la conexion, y herramientas como **fail2ban** banean automaticamente IPs que muestran patrones de ataque de fuerza bruta, monitoreando los logs de autenticacion.

## Modelo mental

Pensa el hardening de SSH como reforzar la puerta principal de un edificio despues de dejarla con una cerradura basica por defecto: quitar la posibilidad de entrar con una llave facil de copiar (contraseña), quitar el acceso directo del dueño del edificio completo (root) por esa puerta especifica, y poner una camara que banea automaticamente a cualquiera que intente forzar la cerradura repetidamente (fail2ban).

## Ejemplo base

```
# /etc/ssh/sshd_config
PasswordAuthentication no
PermitRootLogin no
MaxAuthTries 3
AllowUsers juber deploy
Port 2222              # opcional, reduce ruido de escaneos automatizados
```

```bash
sudo systemctl restart sshd     # aplica los cambios de configuracion

# fail2ban: banea automaticamente IPs con intentos de fuerza bruta
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd    # ver IPs actualmente baneadas
```

## Como se usa en un proyecto real

1. Se aplica `PasswordAuthentication no` y `PermitRootLogin no` en todo servidor con IP publica, como configuracion minima indispensable, nunca opcional.
2. Se prueba siempre la conexion por clave **antes** de deshabilitar autenticacion por contraseña, verificando acceso funcional para no quedar bloqueado del propio servidor.
3. Se instala fail2ban en servidores publicos para mitigar automaticamente ataques de fuerza bruta persistentes, sin necesitar intervencion manual constante.
4. Se restringe `AllowUsers` a los usuarios especificos que realmente necesitan acceso SSH, en vez de dejarlo abierto a cualquier usuario del sistema.
5. Se mantiene siempre un acceso de respaldo (consola fuera de banda del proveedor cloud) antes de aplicar cambios criticos de SSH, previniendo quedar completamente bloqueado si algo sale mal.

## Criterio junior

Un junior debe poder identificar las configuraciones minimas de hardening (`PasswordAuthentication no`, `PermitRootLogin no`) y entender por que son importantes.

## Criterio mid

Un mid debe aplicar hardening de SSH de forma segura (probando acceso por clave antes de deshabilitar contraseña), e instalar fail2ban para mitigar ataques de fuerza bruta.

## Criterio senior

Un senior debe estandarizar la configuracion de hardening de SSH en toda la infraestructura del equipo, auditando periodicamente que servidores cumplen ese estandar, y diseñando el proceso seguro de aplicar cambios sin riesgo de bloqueo.

## Errores comunes

- Deshabilitar autenticacion por contraseña sin haber verificado antes que el acceso por clave funciona correctamente, quedando bloqueado del servidor.
- Dejar `PermitRootLogin yes` (el default historico en muchas distribuciones), exponiendo el usuario mas privilegiado del sistema directamente a intentos de ataque.
- No instalar fail2ban (o equivalente) en servidores con IP publica, dejandolos expuestos a ataques de fuerza bruta persistentes sin mitigacion automatica.
- No mantener un acceso de respaldo antes de aplicar cambios criticos de SSH, arriesgando perder acceso completo al servidor si algo sale mal.

## Practica

- [ ] Verifica que tenes acceso funcional por clave SSH antes de continuar con esta practica.
- [ ] Aplica `PasswordAuthentication no` y `PermitRootLogin no` en un servidor de practica.
- [ ] Instala y configura fail2ban, verificando que monitorea correctamente los intentos fallidos de SSH.
- [ ] Restringe `AllowUsers` a un conjunto especifico de usuarios.
- [ ] Documenta un checklist de hardening de SSH que aplicarias a cualquier servidor nuevo con IP publica.

## Preguntas de repaso

- Por que deshabilitar `PasswordAuthentication` es una de las medidas de seguridad mas importantes para un servidor con IP publica?
- Que verificarias antes de deshabilitar autenticacion por contraseña en un servidor remoto?
- Que hace fail2ban y como complementa el hardening basico de SSH?
- Que acceso de respaldo mantendrias antes de aplicar cambios criticos de configuracion SSH?
