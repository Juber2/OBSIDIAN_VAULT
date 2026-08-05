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

# 04 - Proyecto Pipeline de Monitoreo y Alertas

## Que vas a construir

Un sistema de monitoreo activo del servidor de los proyectos anteriores, aplicando los capitulos de Performance y Troubleshooting, y Administracion de Servidores.

## Alcance funcional

- Script de monitoreo que revisa CPU, memoria (interpretando "available" correctamente), disco, y estado de servicios criticos.
- Ejecucion periodica automatizada via cron, con logging estructurado.
- Alertas reales (email o webhook, no solo logs pasivos) ante condiciones anomalas: servicio caido, disco por encima de un umbral, memoria bajo presion real.
- Revision de `dmesg` incluida, buscando especificamente eventos de OOM Killer.
- Documentacion de los umbrales elegidos y su justificacion.

## Checklist tecnica

- [ ] El script interpreta correctamente memoria "available" (no "free") y load average relativo al numero de nucleos.
- [ ] Se verifica el estado de cada servicio critico con `systemctl is-active`, no solo asumiendo que esta corriendo.
- [ ] Una condicion anomala simulada (servicio detenido a proposito) dispara una alerta real verificable.
- [ ] El script revisa `dmesg` en busca de eventos de OOM Killer recientes.
- [ ] Los umbrales de alerta estan documentados con su justificacion (no valores arbitrarios sin explicacion).

## Criterio de exito

El proyecto se considera completo cuando, al simular al menos dos condiciones anomalas distintas (un servicio caido y un disco cerca del limite), el sistema de monitoreo genera una alerta real y verificable en cada caso, sin falsos positivos por mala interpretacion de metricas (como confundir memoria cacheada con memoria agotada).

## Errores comunes a evitar

- Generar solo logs pasivos sin ninguna alerta activa, dependiendo de que alguien revise manualmente.
- Interpretar mal las metricas de memoria, generando falsas alarmas constantes que el equipo terminaria ignorando.
- No probar realmente que las alertas se disparan simulando condiciones anomalas, confiando solo en que el codigo "deberia funcionar".

## Extension opcional

Integrar las metricas recolectadas con una herramienta de monitoreo mas robusta (Prometheus + Grafana), conectando con lo visto en el capitulo de Monitoreo del libro de Docker para metricas de containers especificamente.
