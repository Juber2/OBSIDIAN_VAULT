---
tags:
  - Docker
  - DevOps
  - Docker-Redes
Creado: 2026-07-28
Relacionado:
  - DOCKER
---

# Redes

## Proposito del capitulo

Este capitulo cubre las redes en Docker: la red bridge default y sus limitaciones, redes custom con DNS interno, los modos host y none, la exposicion correcta de puertos, y patrones de comunicacion resiliente entre containers dependientes.

**Nivel esperado:** Junior a Mid.

## Temas

| Orden | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Bridge Network por Defecto | Pendiente | [[01 - Bridge Network por Defecto]] |
| 2 | Redes Custom y DNS Interno | Pendiente | [[02 - Redes Custom y DNS Interno]] |
| 3 | Host y None Network | Pendiente | [[03 - Host y None Network]] |
| 4 | Exposicion de Puertos | Pendiente | [[04 - Exposicion de Puertos]] |
| 5 | Comunicacion entre Containers | Pendiente | [[05 - Comunicacion entre Containers]] |

## Como cerrar este capitulo

- [ ] Lei cada tema como capitulo, no solo como checklist.
- [ ] Cree una red custom y conecte containers que se comunican por nombre.
- [ ] Puedo explicar la diferencia entre `EXPOSE` y `-p` sin dudar.
- [ ] Configure un health check real usado por `depends_on: condition: service_healthy`.
- [ ] Tengo notas propias de dudas, decisiones o patrones que quiero repetir.
