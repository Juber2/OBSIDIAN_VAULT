---
tags:
  - Docker
  - DevOps
  - Docker-Orquestacion
Creado: 2026-07-28
Relacionado:
  - Orquestacion Basica
  - DOCKER
---

# 02 - Servicios Replicas y Scaling en Swarm

## Que vas a aprender

En este capitulo vas a profundizar en el modelo de **servicios** de Swarm: replicas, el balanceador de carga interno (routing mesh), actualizaciones rolling con rollback automatico, y como desplegar un stack completo desde un `docker-compose.yml`.

## Explicacion

En Swarm, un **servicio** es la unidad de despliegue: declaras cuantas **replicas** queres de un container, y Swarm se encarga de distribuirlas entre los nodos disponibles del cluster, reiniciando automaticamente cualquier replica que muera. El **routing mesh** hace que cualquier nodo del cluster que reciba trafico en el puerto publicado lo enrute automaticamente hacia una replica sana del servicio, sin importar en que nodo especifico este corriendo.

Las **actualizaciones rolling** permiten desplegar una nueva version reemplazando replicas de a poco (en vez de todas de golpe), con la posibilidad de configurar un **rollback automatico** si el health check de las nuevas replicas falla durante el despliegue, evitando que una version rota reemplace completamente a la version anterior funcional.

## Modelo mental

Pensa las replicas de un servicio como varios cajeros identicos atendiendo en distintas sucursales del mismo banco: si un cajero se enferma (un container muere), el banco (Swarm) automaticamente pone a otro a trabajar sin que el cliente note la diferencia. El routing mesh es como un sistema telefonico central que redirige cualquier llamada entrante hacia cualquier sucursal disponible, sin que el cliente necesite saber a cual sucursal especifica esta llamando.

## Ejemplo base

```yaml
# docker-stack.yml, desplegado con `docker stack deploy`
services:
  api:
    image: mi-registro/api:1.4.0
    deploy:
      replicas: 4
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      restart_policy:
        condition: on-failure
    ports:
      - "3000:3000"
```

```bash
docker stack deploy -c docker-stack.yml mi-app
docker service ls
docker service update --image mi-registro/api:1.5.0 mi-app_api
```

## Como se usa en un proyecto real

1. Se define el numero de replicas segun la capacidad esperada de carga y la tolerancia a fallos deseada, nunca dejando un servicio critico en una unica replica.
2. Se configura `update_config` con `failure_action: rollback` para que un despliegue problematico se revierta automaticamente sin intervencion manual urgente.
3. Se despliega desde un archivo de stack versionado en el repositorio (`docker-stack.yml`), no con comandos manuales sueltos, para mantener trazabilidad.
4. Se monitorea el estado de las replicas (`docker service ps`) despues de cada despliegue para confirmar que la actualizacion se completo correctamente.
5. Se aprovecha el routing mesh para no necesitar un load balancer externo adicional en clusters de tamaño moderado.

## Criterio junior

Un junior debe poder desplegar un servicio con multiples replicas en Swarm y verificar que el routing mesh distribuye el trafico correctamente.

## Criterio mid

Un mid debe configurar actualizaciones rolling con rollback automatico, y diagnosticar el estado de un despliegue usando `docker service ps`.

## Criterio senior

Un senior debe diseñar la politica de replicas y actualizaciones para servicios criticos del sistema, definiendo umbrales de fallo que disparan rollback automatico, y estableciendo el proceso de despliegue estandar via archivos de stack versionados.

## Errores comunes

- Dejar un servicio critico con una unica replica, sin tolerancia a fallos si ese unico container muere.
- No configurar `failure_action: rollback`, dejando que un despliegue problematico quede a medio camino sin reversion automatica.
- Desplegar con comandos manuales sueltos en vez de archivos de stack versionados, perdiendo trazabilidad de que se desplego y cuando.
- No verificar el estado de las replicas despues de un despliegue, asumiendo que todo salio bien sin confirmarlo.

## Practica

- [ ] Despliega un servicio con multiples replicas en un cluster Swarm de practica.
- [ ] Verifica que el routing mesh distribuye correctamente el trafico entre replicas.
- [ ] Configura una actualizacion rolling con rollback automatico ante fallo.
- [ ] Simula un despliegue problematico y confirma que el rollback automatico se activa.
- [ ] Despliega desde un archivo de stack versionado en vez de comandos sueltos.

## Preguntas de repaso

- Que hace el routing mesh de Swarm y por que simplifica el balanceo de carga entre replicas?
- Que configuracion permite que un despliegue rolling se revierta automaticamente ante un fallo?
- Por que un servicio critico nunca deberia dejarse con una unica replica?
- Que ventaja de trazabilidad da desplegar desde un archivo de stack versionado sobre comandos manuales?
