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

# 01 - Docker Swarm Cuando Tiene Sentido

## Que vas a aprender

En este capitulo vas a conocer **Docker Swarm**, el orquestador nativo incluido en Docker Engine, entendiendo sus capacidades reales y el tipo de proyecto donde sigue siendo una opcion razonable frente a Kubernetes.

## Explicacion

Docker Swarm convierte un grupo de maquinas con Docker instalado en un **cluster** donde se pueden desplegar servicios replicados, con balanceo de carga interno, actualizaciones rolling, y auto-recuperacion basica (si un container muere, Swarm lo reinicia en algun nodo disponible). Su gran ventaja es la **simplicidad**: no requiere instalar nada adicional mas alla de Docker mismo, y usa el mismo formato de `docker-compose.yml` (con extensiones para deploy) que ya conoces.

Frente a Kubernetes, Swarm es mucho mas simple de operar pero con un ecosistema y capacidades mucho mas limitadas (menos opciones de auto-escalado avanzado, menos integraciones de terceros, comunidad y desarrollo activo mucho menor en los ultimos años). Por eso, Swarm tiene sentido para equipos pequeños con necesidades moderadas de alta disponibilidad que no justifican la complejidad operativa de Kubernetes, mientras que Kubernetes domina el espacio de sistemas grandes y complejos.

## Modelo mental

Pensa Docker Swarm como un auto familiar confiable y facil de mantener vos mismo: te lleva bien a la mayoria de destinos comunes sin complicaciones. Kubernetes es como un vehiculo industrial con muchisimas mas capacidades y accesorios disponibles, pero que requiere un mecanico especializado (un equipo de plataforma dedicado) para sacarle verdadero provecho.

## Ejemplo base

```bash
docker swarm init                    # convierte el nodo actual en manager del cluster
docker swarm join-token worker       # obtiene el comando para sumar nodos worker

docker service create --name api --replicas 3 -p 3000:3000 mi-registro/api:1.0
docker service scale api=5           # escala a 5 replicas
docker service update --image mi-registro/api:1.1 api   # rolling update
```

## Como se usa en un proyecto real

1. Se evalua Swarm para equipos pequeños/medianos que necesitan alta disponibilidad basica y despliegues rolling sin justificar un equipo de plataforma dedicado a Kubernetes.
2. Se reutiliza el conocimiento ya adquirido de Docker Compose, dado que Swarm usa el mismo formato de archivo con la seccion `deploy:` adicional.
3. Se es realista sobre las limitaciones: Swarm no tiene el mismo nivel de auto-escalado avanzado, operadores de terceros, ni observabilidad del ecosistema que Kubernetes.
4. Se documenta la decision de usar Swarm (o no) como una decision arquitectonica explicita, con el marco visto en el capitulo Senior de NestJS, considerando el tamaño real del equipo y sus necesidades.
5. Se planifica desde el inicio si el crecimiento esperado del sistema eventualmente justificaria migrar a Kubernetes, para no quedar atado a una decision que ya no encaja.

## Criterio junior

Un junior debe entender que es Docker Swarm y poder desplegar un servicio replicado basico en un cluster de practica.

## Criterio mid

Un mid debe realizar actualizaciones rolling y escalado de servicios en Swarm, y comparar sus capacidades con las de Kubernetes para un caso concreto.

## Criterio senior

Un senior debe decidir con criterio entre Swarm y Kubernetes para un proyecto nuevo, considerando el tamaño del equipo, la complejidad operativa aceptable, y las necesidades reales de escalado, evitando adoptar Kubernetes "porque es lo que se usa" sin justificacion real.

## Errores comunes

- Adoptar Kubernetes para un proyecto pequeño que Swarm resolveria con mucha menos complejidad operativa, sin justificacion real de esa complejidad adicional.
- Quedarse en Swarm cuando las necesidades reales de escalado y ecosistema ya superan claramente sus capacidades, sin evaluar migrar a tiempo.
- No considerar el tamaño y experiencia del equipo como factor central de esta decision, evaluando solo capacidades tecnicas en abstracto.
- Asumir que Swarm esta activamente evolucionando al mismo ritmo que Kubernetes, cuando en la practica su desarrollo es mucho mas limitado.

## Practica

- [ ] Inicializa un cluster Swarm local (aunque sea de un solo nodo) y despliega un servicio replicado.
- [ ] Practica escalar y hacer un rolling update de ese servicio.
- [ ] Compara conceptualmente las capacidades de Swarm y Kubernetes para un caso hipotetico de tu contexto.
- [ ] Evalua si tu equipo actual justificaria la complejidad de Kubernetes o si Swarm alcanzaria.
- [ ] Documenta el criterio que usarias para decidir entre ambos en un proyecto nuevo.

## Preguntas de repaso

- Que ventaja principal tiene Docker Swarm sobre Kubernetes en terminos de simplicidad operativa?
- Que limitaciones tiene Swarm frente a Kubernetes en terminos de ecosistema y capacidades avanzadas?
- Que factores del equipo (no solo tecnicos) deberian influir en la eleccion entre Swarm y Kubernetes?
- En que momento un proyecto que empezo con Swarm podria justificar migrar a Kubernetes?
