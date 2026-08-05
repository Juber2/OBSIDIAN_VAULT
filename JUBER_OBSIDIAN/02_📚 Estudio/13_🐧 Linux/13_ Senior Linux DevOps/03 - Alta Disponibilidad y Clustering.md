---
tags:
  - Linux
  - DevOps
  - Linux-Senior
Creado: 2026-07-28
Relacionado:
  - Senior Linux DevOps
  - LINUX
---

# 03 - Alta Disponibilidad y Clustering

## Que vas a aprender

En este capitulo vas a aprender los fundamentos de **alta disponibilidad (HA)** a nivel de infraestructura Linux: **load balancers**, **failover**, y **health checks** de infraestructura, conectando con lo ya visto en Orquestacion del libro de Docker desde una capa mas baja.

## Explicacion

Alta disponibilidad significa diseñar un sistema para que la falla de un componente individual no cause una interrupcion completa del servicio. Un **load balancer** (HAProxy, Nginx como ya visto, o soluciones cloud gestionadas) distribuye trafico entre multiples servidores backend, y cuando se combina con **health checks** activos (verificando periodicamente que cada backend responde correctamente, el mismo principio ya visto para containers en el libro de Docker), automaticamente saca de rotacion a un servidor que dejo de responder, dirigiendo trafico solo a los que siguen sanos.

**Failover** es el mecanismo que permite que un servicio critico (como una base de datos, o el load balancer mismo) tenga un respaldo listo para tomar el control si el primario falla, minimizando el tiempo de inactividad. Herramientas como **Keepalived** implementan failover de IP virtual (VRRP): dos servidores comparten una IP "flotante" que automaticamente se mueve al servidor de respaldo si el primario deja de responder, sin necesitar cambiar configuracion de DNS ni de clientes.

## Modelo mental

Pensa un load balancer con health checks como un despachador de taxis que solo envia pasajeros a taxis que confirmaron estar disponibles y funcionando, sacando automaticamente de la rotacion a cualquiera que deje de responder. El failover con IP virtual es como tener un numero de telefono unico de emergencias que, sin que el que llama lo note, automaticamente redirige la llamada a un segundo operador de respaldo si el primero no contesta.

## Ejemplo base

```
# HAProxy: balanceo con health check activo (configuracion conceptual)
backend servidores_api
    balance roundrobin
    option httpchk GET /health
    server api1 10.0.0.10:3000 check
    server api2 10.0.0.11:3000 check
```

```bash
# Keepalived: IP virtual compartida entre dos servidores (VRRP)
# El servidor con mayor "priority" activo toma la IP virtual
# Si el primario falla, el secundario la toma automaticamente
```

## Como se usa en un proyecto real

1. Se coloca un load balancer con health checks reales delante de cualquier servicio que necesite alta disponibilidad real, siguiendo el mismo principio ya visto para containers en el libro de Docker.
2. Se implementa failover de IP virtual para componentes de infraestructura criticos que no pueden depender de un unico punto de fallo (el load balancer mismo, una base de datos primaria).
3. Se diseñan health checks reales (que verifican dependencias criticas, no solo que el proceso responde) en cada capa: aplicacion, load balancer, e infraestructura.
4. Se conecta este conocimiento con el capitulo de Orquestacion Basica del libro de Docker: Swarm y Kubernetes implementan versiones mas sofisticadas de estos mismos principios a nivel de containers.
5. Se documenta el diseño de alta disponibilidad de sistemas criticos, incluyendo que componentes tienen redundancia y cuales siguen siendo un punto unico de fallo aceptado.

## Criterio junior

Un junior debe entender conceptualmente que hace un load balancer y por que los health checks son necesarios para que funcione correctamente.

## Criterio mid

Un mid debe configurar un load balancer basico con health checks reales, y entender el concepto de failover de IP virtual.

## Criterio senior

Un senior debe diseñar la arquitectura completa de alta disponibilidad de un sistema, identificando y eliminando puntos unicos de fallo criticos, coordinando con las estrategias de orquestacion de containers ya vistas en el libro de Docker.

## Errores comunes

- Configurar un load balancer sin health checks reales, siguiendo enviando trafico a servidores caidos.
- No implementar failover para componentes de infraestructura criticos, dejando un unico punto de fallo sin redundancia.
- Confundir "tener un load balancer" con "tener alta disponibilidad real", sin verificar que efectivamente distribuye y saca de rotacion correctamente.
- No documentar que puntos unicos de fallo siguen existiendo conscientemente en un sistema, dejando esa informacion solo en la cabeza de quien lo diseño.

## Practica

- [ ] Configura un load balancer basico (HAProxy o Nginx) con health check real hacia dos backends de practica.
- [ ] Simula la caida de un backend y verifica que el load balancer lo saca de rotacion automaticamente.
- [ ] Investiga como funciona Keepalived para failover de IP virtual entre dos servidores.
- [ ] Identifica los puntos unicos de fallo de un sistema real que conozcas.
- [ ] Documenta el diseño de alta disponibilidad (o su ausencia consciente) de ese sistema.

## Preguntas de repaso

- Que rol cumplen los health checks en el funcionamiento correcto de un load balancer?
- Que problema resuelve el failover de IP virtual con Keepalived?
- Como se relacionan estos principios de HA con la orquestacion de containers vista en el libro de Docker?
- Como identificarias los puntos unicos de fallo de un sistema real?
