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

# 03 - Proyecto Hardening de Seguridad de un Servidor

## Que vas a construir

Un ejercicio completo de hardening sobre el servidor de los proyectos anteriores, aplicando de punta a punta el capitulo de Seguridad.

## Alcance funcional

- SSH completamente hardened: sin contraseña, sin root login, con fail2ban activo.
- Sudo configurado con reglas granulares (no acceso total innecesario) para cada usuario de servicio.
- SELinux o AppArmor activo y en modo enforcing/enforce, con cualquier bloqueo legitimo ajustado correctamente (no deshabilitado).
- Actualizaciones automaticas de seguridad configuradas, limitadas a parches.
- Auditoria basica configurada (auditd o revision periodica de logs de autenticacion) sobre al menos un archivo critico.

## Checklist tecnica

- [ ] `PasswordAuthentication no` y `PermitRootLogin no` verificados y funcionando (con acceso por clave probado antes).
- [ ] fail2ban activo, verificado con un intento de fuerza bruta simulado que resulte en un baneo real.
- [ ] SELinux/AppArmor en modo enforcing/enforce, sin ningun ajuste que lo deshabilite completamente.
- [ ] Actualizaciones automaticas de seguridad configuradas y verificadas (revisando la configuracion, no solo asumiendo que funciona).
- [ ] Al menos una regla de auditoria activa sobre un archivo critico, con un evento de prueba registrado exitosamente.

## Criterio de exito

El proyecto se considera completo cuando existe documentacion (un mini-ADR) de cada decision de hardening aplicada, con evidencia de que cada control realmente funciona (no solo que esta configurado), y sin ningun mecanismo de seguridad deshabilitado por conveniencia durante el proceso.

## Errores comunes a evitar

- Deshabilitar SELinux/AppArmor "temporalmente" para resolver un problema y olvidar reactivarlo.
- Aplicar hardening de SSH sin haber verificado antes que el acceso por clave funciona, arriesgando quedar bloqueado.
- No probar realmente que fail2ban banea (solo confiar en que esta "instalado").

## Extension opcional

Documentar un proceso de respuesta ante un evento de seguridad detectado (por ejemplo, multiples intentos de fuerza bruta detectados por fail2ban), como puente hacia una mentalidad operativa mas madura.
