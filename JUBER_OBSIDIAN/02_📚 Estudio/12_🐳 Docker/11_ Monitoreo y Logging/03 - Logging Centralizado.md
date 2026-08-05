---
tags:
  - Docker
  - DevOps
  - Docker-Monitoreo
Creado: 2026-07-28
Relacionado:
  - Monitoreo y Logging
  - DOCKER
---

# 03 - Logging Centralizado

## Que vas a aprender

En este capitulo vas a aprender a enviar logs de containers a un sistema **centralizado**, indispensable en cuanto un sistema corre en mas de un servidor: patrones con ELK/Loki, y el rol de un agente recolector como Fluentd o Fluent Bit.

## Explicacion

Con un solo servidor, `docker logs` puede alcanzar para diagnosticar problemas. En cuanto el sistema corre en **multiples servidores o containers efimeros** (que se recrean y desaparecen), buscar en logs dispersos servidor por servidor se vuelve inviable. El patron estandar es usar un **agente recolector** (Fluentd, Fluent Bit, o el driver de logging nativo de Docker configurado directamente) que envia los logs de cada container a un destino centralizado: **ELK** (Elasticsearch + Logstash + Kibana, mas pesado pero muy potente) o **Loki** (de Grafana, mas liviano, diseñado especificamente para logs con la misma filosofia que Prometheus para metricas).

Independientemente de la herramienta elegida, el valor real aparece cuando los logs incluyen **contexto estructurado** (ver el capitulo de Observabilidad en el libro de NestJS: logging en JSON con request ID de correlacion), permitiendo buscar y filtrar de forma precisa en vez de hacer grep de texto libre sobre volumenes enormes de logs.

## Modelo mental

Pensa el logging centralizado como pasar de tener el diario de cada sucursal de una cadena de tiendas guardado localmente en cada sucursal, a tener todos los diarios enviados automaticamente a una oficina central que los organiza, indexa, y permite buscar "todo lo que paso en cualquier sucursal el martes a las 3pm" con una sola consulta.

## Ejemplo base

```yaml
# docker-compose.yml: enviar logs directo a Loki via driver nativo
services:
  api:
    image: mi-imagen
    logging:
      driver: loki
      options:
        loki-url: "http://loki:3100/loki/api/v1/push"
```

```yaml
# Alternativa: Fluent Bit como agente recolector, leyendo logs y reenviandolos
services:
  fluent-bit:
    image: fluent/fluent-bit
    volumes:
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - ./fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
```

## Como se usa en un proyecto real

1. Se centraliza el logging en cuanto el sistema corre en mas de un servidor, sin esperar a que la falta de esto ya cause un incidente dificil de diagnosticar.
2. Se emiten logs en formato estructurado (JSON) desde la aplicacion, no texto libre, para que el sistema centralizado pueda indexarlos y filtrarlos de forma precisa.
3. Se elige Loki cuando ya se usa el ecosistema de Grafana/Prometheus (integracion natural), o ELK cuando se necesita busqueda de texto completo mas potente o ya existe experiencia con ese stack.
4. Se define una politica de retencion de logs centralizados, balanceando costo de almacenamiento contra necesidad de historial para auditorias o diagnostico de incidentes pasados.
5. Se correlacionan logs centralizados con metricas (ver tema anterior) y traces cuando existen, siguiendo los tres pilares de observabilidad.

## Criterio junior

Un junior debe entender por que buscar logs servidor por servidor se vuelve inviable en un sistema con multiples servidores, y conocer que existen soluciones centralizadas.

## Criterio mid

Un mid debe configurar el envio de logs de containers a un sistema centralizado (Loki o ELK) usando un driver nativo o un agente recolector.

## Criterio senior

Un senior debe diseñar la estrategia de logging centralizado de toda la organizacion, eligiendo el stack apropiado, definiendo politicas de retencion, y asegurando que los logs estructurados permitan diagnostico rapido de incidentes reales en produccion.

## Errores comunes

- Depender de logs locales por servidor en un sistema que ya corre en multiples servidores, haciendo el diagnostico de incidentes lento y frustrante.
- Enviar logs en texto libre sin estructura al sistema centralizado, perdiendo la capacidad de filtrar y buscar de forma precisa.
- No definir una politica de retencion, dejando que el costo de almacenamiento de logs crezca sin control con el tiempo.
- No correlacionar logs con un request ID (ver capitulo de Observabilidad en NestJS), dificultando seguir el rastro completo de una operacion especifica.

## Practica

- [ ] Configura el envio de logs de un container a un sistema centralizado (Loki o ELK) de practica.
- [ ] Verifica que puedes buscar y filtrar logs usando el sistema centralizado en vez de `docker logs` directo.
- [ ] Emite logs en formato JSON estructurado desde una aplicacion de practica.
- [ ] Define una politica de retencion de logs centralizados para tu contexto.
- [ ] Investiga como correlacionarias logs centralizados con metricas y traces en un stack de observabilidad completo.

## Preguntas de repaso

- Por que buscar logs servidor por servidor se vuelve inviable en un sistema con multiples servidores?
- Que ventaja da el logging estructurado (JSON) sobre texto libre en un sistema centralizado?
- Cuando elegirias Loki sobre ELK, o viceversa, para tu stack de observabilidad?
- Que factores considerarias al definir una politica de retencion de logs centralizados?
