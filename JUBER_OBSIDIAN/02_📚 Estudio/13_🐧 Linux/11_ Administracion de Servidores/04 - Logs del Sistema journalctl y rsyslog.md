---
tags:
  - Linux
  - Administracion
  - Linux-Servidores
Creado: 2026-07-28
Relacionado:
  - Administracion de Servidores
  - LINUX
---

# 04 - Logs del Sistema journalctl y rsyslog

## Que vas a aprender

En este capitulo vas a profundizar en el sistema de logging de Linux a nivel de sistema operativo: **journalctl** (el journal de systemd, ya usado en capitulos anteriores) en profundidad, y **rsyslog** como sistema de logging tradicional aun ampliamente usado, conectando con el logging centralizado del libro de Docker.

## Explicacion

`journalctl` (el visor del journal de systemd) centraliza logs de todos los servicios administrados por systemd en un unico lugar, con filtrado potente: por servicio (`-u`), por rango de tiempo (`--since`, `--until`), por prioridad (`-p err` solo errores), y en formato JSON estructurado (`-o json`) para integracion con herramientas externas. A diferencia de archivos de log tradicionales dispersos en `/var/log/`, el journal es binario y estructurado, con metadata rica (PID, unidad de systemd, prioridad) automaticamente disponible en cada entrada.

**rsyslog** (o su alternativa `syslog-ng`) sigue siendo relevante para aplicaciones que no corren bajo systemd, o para **reenviar logs a un servidor centralizado** via el protocolo syslog estandar, un mecanismo que muchas herramientas y appliances de red todavia usan. En la practica, muchos sistemas usan ambos: journalctl para logs locales estructurados de servicios systemd, y rsyslog configurado para reenviar (o recibir) logs hacia/desde un sistema centralizado.

## Modelo mental

Pensa `journalctl` como un archivo digital centralizado y bien organizado de todo lo que paso en el sistema, con capacidad de busqueda avanzada por multiples criterios simultaneos. `rsyslog` es como el sistema de correo tradicional entre edificios: cada edificio (servidor) puede enviar copias de sus registros a una oficina central (servidor de logs centralizado), siguiendo un protocolo estandar que casi cualquier sistema, viejo o nuevo, entiende.

## Ejemplo base

```bash
journalctl -u nginx --since "1 hour ago"     # logs de un servicio, ultima hora
journalctl -p err -b                          # solo errores, desde el boot actual
journalctl -f                                  # sigue el journal completo en tiempo real
journalctl --disk-usage                        # cuanto espacio ocupa el journal actualmente
sudo journalctl --vacuum-time=7d                # limita el journal a los ultimos 7 dias

# rsyslog: reenviar logs a un servidor centralizado
# /etc/rsyslog.conf
*.* @@servidor-logs.interno:514    # @@ = TCP, @ = UDP
```

## Como se usa en un proyecto real

1. Se usa `journalctl -u servicio --since` como primera herramienta de diagnostico ante cualquier problema de un servicio administrado por systemd.
2. Se configura `--vacuum-time` o `--vacuum-size` para limitar el crecimiento del journal, evitando que ocupe espacio de disco indefinidamente (conectando con lo visto en el capitulo de FHS sobre monitorear `/var`).
3. Se configura rsyslog para reenviar logs a un servidor centralizado en infraestructura con multiples servidores, siguiendo el mismo principio de logging centralizado ya visto en el libro de Docker.
4. Se usa `journalctl -o json` para integrar logs del sistema con herramientas de analisis externas que esperan formato estructurado.
5. Se combina el logging de sistema (journalctl/rsyslog) con el logging de aplicacion (visto en el capitulo de Observabilidad del libro de NestJS) para tener visibilidad completa de un incidente, desde la capa de sistema operativo hasta la aplicacion.

## Criterio junior

Un junior debe poder usar `journalctl -u servicio` y `journalctl -f` para revisar logs de un servicio especifico.

## Criterio mid

Un mid debe filtrar logs por prioridad y rango de tiempo, y gestionar el tamaño del journal con `--vacuum-time`/`--vacuum-size`.

## Criterio senior

Un senior debe diseñar la estrategia de logging de sistema de toda la infraestructura, centralizando logs con rsyslog hacia un sistema comun, coordinado con el logging de aplicacion y de containers ya visto en libros anteriores.

## Errores comunes

- No limitar el tamaño del journal, dejando que ocupe espacio de disco indefinidamente con el tiempo.
- No usar filtros de journalctl (`-u`, `--since`, `-p`) y en cambio revisar logs sin ningun criterio, perdiendo tiempo buscando manualmente.
- No centralizar logs de sistema en infraestructura con multiples servidores, dificultando correlacionar eventos entre distintos sistemas.
- No coordinar el logging de sistema con el logging de aplicacion, perdiendo contexto completo al diagnosticar un incidente.

## Practica

- [ ] Revisa los logs de un servicio especifico con `journalctl -u` filtrando por un rango de tiempo.
- [ ] Filtra solo errores con `journalctl -p err`.
- [ ] Verifica el espacio ocupado por el journal y limitalo con `--vacuum-time`.
- [ ] Investiga como configurarias rsyslog para reenviar logs a un servidor centralizado.
- [ ] Documenta como combinarias logging de sistema con logging de aplicacion para diagnosticar un incidente completo.

## Preguntas de repaso

- Que ventaja da el journal de systemd sobre archivos de log tradicionales dispersos en `/var/log/`?
- Por que es importante limitar el tamaño del journal con `--vacuum-time` o `--vacuum-size`?
- Que rol cumple rsyslog en infraestructura con multiples servidores?
- Como combinarias logging de sistema con logging de aplicacion para un diagnostico completo de un incidente?
