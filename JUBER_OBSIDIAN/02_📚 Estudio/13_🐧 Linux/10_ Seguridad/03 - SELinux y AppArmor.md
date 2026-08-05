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

# 03 - SELinux y AppArmor

## Que vas a aprender

En este capitulo vas a aprender sobre los sistemas de **Mandatory Access Control (MAC)** de Linux: **SELinux** (RedHat/CentOS/Fedora) y **AppArmor** (Ubuntu/Debian), que van mas alla de los permisos rwx tradicionales para confinar procesos con politicas de seguridad muy granulares.

## Explicacion

Los permisos Unix tradicionales (rwx, ya vistos) son **Discretionary Access Control (DAC)**: el dueño de un archivo decide sus permisos, y un proceso comprometido corriendo con los privilegios de ese usuario puede hacer lo que ese usuario podria hacer. **SELinux** y **AppArmor** agregan una capa adicional de **Mandatory Access Control**: incluso si un proceso tiene permisos rwx suficientes segun el modelo tradicional, una politica MAC puede **igual prohibirle** esa accion si no esta explicitamente permitida por su perfil de seguridad, limitando drasticamente el impacto de una aplicacion comprometida (por ejemplo, evitando que un servidor web comprometido pueda leer `/etc/shadow` aunque tecnicamente el usuario bajo el que corre tenga, por error de configuracion, permisos de lectura ahi).

SELinux usa **contextos de seguridad** (etiquetas asignadas a archivos y procesos) y politicas muy detalladas, siendo mas potente pero tambien mas complejo de administrar. AppArmor usa **perfiles basados en rutas de archivo**, mas simple de entender y configurar, aunque algo menos granular que SELinux.

## Modelo mental

Pensa los permisos rwx tradicionales como el sistema de llaves normales de un edificio: si tenes la llave (permiso), podes entrar. SELinux/AppArmor son como un guardia de seguridad adicional parado en cada puerta que, ademas de verificar que tengas la llave correcta, revisa una lista separada de "que tiene permitido hacer especificamente esta persona en este contexto", pudiendo negar el paso incluso a alguien con la llave correcta si su perfil de seguridad no lo autoriza para esa accion especifica.

## Ejemplo base

```bash
# SELinux (RedHat/CentOS)
getenforce                      # modo actual: Enforcing, Permissive, o Disabled
sudo setenforce 0                # cambia a Permissive temporalmente (solo registra, no bloquea)
ls -Z /var/www/html               # muestra el contexto SELinux de archivos
sudo semanage fcontext -a -t httpd_sys_content_t "/otra/ruta(/.*)?"   # ajusta contexto

# AppArmor (Ubuntu/Debian)
sudo aa-status                   # perfiles activos y su modo (enforce/complain)
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx    # modo complain: solo registra, no bloquea
```

## Como se usa en un proyecto real

1. Se mantiene SELinux/AppArmor activo (`Enforcing`/`enforce`) en produccion, resistiendo la tentacion de deshabilitarlo para "resolver rapido" un problema de permisos sin entender la causa real.
2. Se usa el modo `Permissive`/`complain` temporalmente durante troubleshooting, para identificar que reglas estarian bloqueando algo sin bloquearlo realmente, antes de ajustar la politica correctamente.
3. Se ajustan contextos SELinux o perfiles AppArmor especificamente para las necesidades de una aplicacion propia, en vez de deshabilitar la proteccion completa del sistema.
4. Se investigan los logs de auditoria (`/var/log/audit/audit.log` en SELinux, `dmesg`/`journalctl` para AppArmor) cuando algo falla de forma inesperada y se sospecha que MAC lo esta bloqueando.
5. Se documenta cualquier ajuste de politica no obvio realizado para una aplicacion especifica, facilitando el mantenimiento futuro.

## Criterio junior

Un junior debe entender que SELinux/AppArmor son una capa adicional de seguridad mas alla de los permisos tradicionales, y saber verificar su estado actual.

## Criterio mid

Un mid debe diagnosticar cuando SELinux/AppArmor esta bloqueando una accion legitima, usando modo permissive/complain para identificar la regla necesaria sin deshabilitar la proteccion completa.

## Criterio senior

Un senior debe mantener SELinux/AppArmor activo en toda la infraestructura de produccion, ajustando politicas especificas para aplicaciones propias en vez de deshabilitar la proteccion, y liderar la resistencia cultural del equipo a "simplemente apagarlo" como solucion rapida.

## Errores comunes

- Deshabilitar SELinux/AppArmor completamente ante el primer problema de permisos, perdiendo una capa de seguridad valiosa en vez de entender y resolver la causa real.
- No revisar logs de auditoria especificos de SELinux/AppArmor cuando algo falla de forma inexplicable, asumiendo que el problema es de otra naturaleza.
- No documentar ajustes de politica realizados para aplicaciones propias, dificultando el mantenimiento futuro por otro administrador.
- Confundir un problema de permisos rwx tradicionales con un bloqueo de MAC, perdiendo tiempo diagnosticando en la capa equivocada.

## Practica

- [ ] Verifica el estado actual de SELinux o AppArmor en tu sistema (segun la distribucion que uses).
- [ ] Si tenes SELinux, revisa el contexto de seguridad de algunos archivos con `ls -Z`.
- [ ] Provoca un bloqueo de MAC a proposito (por ejemplo, moviendo un archivo web a una ubicacion no esperada) y diagnostica el problema en modo permissive/complain.
- [ ] Ajusta una politica especifica para resolver ese bloqueo sin deshabilitar la proteccion completa.
- [ ] Documenta el ajuste de politica realizado para referencia futura.

## Preguntas de repaso

- Que diferencia hay entre Discretionary Access Control (permisos rwx) y Mandatory Access Control (SELinux/AppArmor)?
- Por que deshabilitar SELinux/AppArmor completamente es una mala practica de seguridad, aunque resuelva un problema rapido?
- Como usarias el modo permissive/complain para diagnosticar un bloqueo sin deshabilitar la proteccion?
- Donde revisarias logs especificos de SELinux o AppArmor ante un fallo inexplicable de una aplicacion?
