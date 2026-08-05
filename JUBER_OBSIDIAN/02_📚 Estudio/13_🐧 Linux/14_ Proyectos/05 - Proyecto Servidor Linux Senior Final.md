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

# 05 - Proyecto Servidor Linux Senior Final

## Que vas a construir

El proyecto integrador del libro: tomar el servidor hardened y monitoreado de los proyectos anteriores, y llevarlo al nivel de infraestructura reproducible del capitulo Senior, con la aplicacion corriendo dockerizada (conectando explicitamente con el libro de Docker).

## Alcance funcional

- Toda la configuracion del servidor (paquetes, servicios, firewall, usuarios) declarada en un playbook de Ansible reproducible, no configurada manualmente.
- La aplicacion corriendo dentro de containers Docker (aplicando el libro de Docker completo), en vez de directamente sobre el sistema operativo.
- Nginx del host actuando como reverse proxy hacia los containers, aplicando el mismo patron de arquitectura ya visto en ambos libros.
- Backup automatizado de volumenes de Docker y de configuracion del host coordinados en un unico proceso.
- Documentacion completa: README de setup reproducible via Ansible, y al menos dos ADRs de decisiones de infraestructura tomadas.

## Checklist tecnica

- [ ] Ejecutar el playbook de Ansible sobre un servidor limpio reproduce exactamente la misma configuracion, verificado con `--check` mostrando cero cambios en una segunda ejecucion.
- [ ] La aplicacion corre en containers Docker, con las buenas practicas de ese libro aplicadas (multi-stage, usuario no-root, health checks).
- [ ] El firewall del host y las redes de Docker estan coordinados sin conflictos ni exposicion no intencional.
- [ ] El backup cubre tanto volumenes de Docker como configuracion critica del host, en un unico proceso documentado.
- [ ] Existe monitoreo activo (del proyecto anterior) cubriendo tanto el host como los containers.

## Criterio de exito

El proyecto se considera completo cuando se puede destruir el servidor completo y reconstruirlo desde cero ejecutando unicamente el playbook de Ansible y desplegando los containers Docker, sin ningun paso de configuracion manual adicional, obteniendo un sistema funcionalmente identico al original.

## Errores comunes a evitar

- Dejar cualquier paso de configuracion manual fuera del playbook de Ansible, rompiendo la reproducibilidad completa.
- No coordinar el firewall del host con las reglas que Docker modifica automaticamente, generando exposicion no intencional.
- Tratar el backup de containers y de host como procesos completamente separados sin ninguna coordinacion.

## Cierre del libro

Completar este proyecto es la señal de haber recorrido el camino completo de este libro: desde entender el kernel y las distribuciones hasta operar infraestructura Linux reproducible, segura, monitoreada y ejecutando aplicaciones containerizadas con criterio senior. Este es tambien el punto donde los tres libros de este vault ([[NESTJS|NestJS]], [[DOCKER|Docker]], y [[LINUX|Linux]]) convergen en un unico sistema real: una aplicacion NestJS, empaquetada con Docker, corriendo sobre infraestructura Linux administrada como codigo. El siguiente paso natural es aplicar exactamente este mismo criterio a un servidor de trabajo real, como el que aloja StaffTrainingPlatform.
