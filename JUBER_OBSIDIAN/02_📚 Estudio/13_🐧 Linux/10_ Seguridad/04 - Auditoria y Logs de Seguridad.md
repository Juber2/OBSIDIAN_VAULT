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

# 04 - Auditoria y Logs de Seguridad

## Que vas a aprender

En este capitulo vas a aprender a auditar la actividad de seguridad de un sistema Linux: logs de autenticacion, **auditd** para auditoria detallada de eventos del sistema, y como detectar actividad sospechosa antes de que se convierta en un incidente.

## Explicacion

Los logs de autenticacion (`/var/log/auth.log` en Debian/Ubuntu, `/var/log/secure` en RedHat/derivados) registran intentos de login (exitosos y fallidos), uso de sudo, y cambios de usuario, siendo la primera fuente de evidencia ante actividad sospechosa. **auditd** es un sistema mas profundo de auditoria del kernel, capaz de registrar eventos especificos configurables: quien accedio a un archivo critico, que comandos se ejecutaron, cambios en la configuracion del sistema, ofreciendo trazabilidad mucho mas detallada que los logs de aplicacion estandar.

Patrones sospechosos que un administrador deberia poder identificar revisando estos logs: multiples intentos fallidos de login desde la misma IP en poco tiempo (fuerza bruta), logins exitosos desde ubicaciones geograficas o IPs inusuales, uso de sudo fuera del horario habitual de trabajo del equipo, o modificaciones inesperadas a archivos criticos del sistema (`/etc/passwd`, `/etc/sudoers`).

## Modelo mental

Pensa los logs de autenticacion como el registro de entrada de un edificio (quien entro, cuando, con que credencial), y auditd como camaras de seguridad adicionales apuntando a las areas mas sensibles del edificio (la boveda, la sala de servidores), registrando exactamente quien toco que y cuando, con mucho mas detalle del que da simplemente saber quien entro al edificio.

## Ejemplo base

```bash
grep "Failed password" /var/log/auth.log | tail -20   # intentos fallidos recientes
grep "sudo" /var/log/auth.log | grep "COMMAND"          # comandos ejecutados con sudo

# auditd: monitorear cambios en un archivo critico
sudo auditctl -w /etc/passwd -p wa -k cambios_passwd
sudo ausearch -k cambios_passwd    # busca eventos registrados con esa etiqueta

sudo aureport --auth --summary      # resumen de eventos de autenticacion via auditd
```

## Como se usa en un proyecto real

1. Se revisan logs de autenticacion periodicamente (o se integran con un sistema de alertas automatizado) buscando patrones de intentos fallidos repetidos, complementando fail2ban visto en el tema anterior.
2. Se configura auditd para monitorear archivos criticos (`/etc/passwd`, `/etc/sudoers`, binarios de sistema sensibles) en servidores de alta sensibilidad.
3. Se centralizan logs de auditoria de multiples servidores (siguiendo el mismo principio de logging centralizado visto en el libro de Docker) para poder correlacionar actividad entre sistemas.
4. Se documenta un proceso de revision regular de logs de seguridad, no solo reactivo ante un incidente ya ocurrido.
5. Se establece retencion adecuada de logs de auditoria, balanceando espacio de almacenamiento contra la necesidad de investigar incidentes que se descubren tiempo despues de ocurridos.

## Criterio junior

Un junior debe poder revisar logs de autenticacion basicos identificando intentos fallidos de login y uso de sudo.

## Criterio mid

Un mid debe configurar auditd para monitorear archivos criticos especificos, y correlacionar eventos de logs para identificar patrones sospechosos.

## Criterio senior

Un senior debe diseñar la estrategia de auditoria de seguridad de toda la infraestructura, centralizando logs para correlacion entre servidores, y estableciendo el proceso de revision regular y respuesta ante hallazgos sospechosos.

## Errores comunes

- No revisar logs de autenticacion regularmente, descubriendo actividad sospechosa solo despues de que ya causo un incidente.
- No configurar auditd para archivos criticos especificos, perdiendo visibilidad detallada de quien modifico configuracion sensible del sistema.
- No centralizar logs de auditoria entre servidores, dificultando correlacionar actividad sospechosa que abarca multiples sistemas.
- No definir una politica de retencion de logs de auditoria, perdiendo evidencia util para investigar incidentes descubiertos tarde.

## Practica

- [ ] Revisa los logs de autenticacion de tu sistema identificando intentos fallidos recientes.
- [ ] Revisa el historial de comandos ejecutados con sudo en los logs.
- [ ] Configura auditd para monitorear cambios en un archivo critico especifico.
- [ ] Investiga como centralizarias logs de auditoria de multiples servidores.
- [ ] Documenta un proceso de revision regular de logs de seguridad para tu infraestructura.

## Preguntas de repaso

- Que patrones en logs de autenticacion indicarian un posible ataque de fuerza bruta?
- Que ventaja adicional da auditd sobre los logs de autenticacion basicos?
- Por que centralizar logs de auditoria entre servidores es importante para detectar actividad sospechosa distribuida?
- Que balance considerarias al definir una politica de retencion de logs de auditoria?
