---
tags:
  - Docker
  - DevOps
  - Docker-Redes
Creado: 2026-07-28
Relacionado:
  - Redes
  - DOCKER
---

# 02 - Redes Custom y DNS Interno

## Que vas a aprender

En este capitulo vas a aprender a crear **redes bridge custom**, que a diferencia de la red default, proveen **resolucion DNS automatica por nombre de container**, siendo el patron estandar para comunicar servicios relacionados.

## Explicacion

Cuando creas una red bridge custom (`docker network create`) y conectas containers a ella, Docker activa automaticamente un servidor DNS interno que resuelve el **nombre del container** (o el nombre del servicio, en Compose) a su IP actual dentro de esa red. Esto significa que un container puede simplemente usar el nombre `db` o `api` como si fuera un hostname real, sin preocuparse por la IP interna, que puede cambiar libremente si el container se recrea.

Esta es la razon por la cual, en Docker Compose (ver capitulo dedicado), los servicios se comunican entre si usando el nombre del servicio como hostname directamente, sin configuracion adicional: Compose crea automaticamente una red custom para el proyecto y conecta todos los servicios a ella.

## Modelo mental

Pensa una red bridge custom como el mismo edificio de departamentos del tema anterior, pero ahora con un portero que mantiene actualizada una lista de "quien vive en que puerta ahora" y te la dice apenas preguntas por el nombre del inquilino, sin que necesites memorizar numeros de puerta que ademas pueden cambiar.

## Ejemplo base

```bash
docker network create mi-red

docker run -d --name db --network mi-red postgres:16
docker run -d --name api --network mi-red -e DB_HOST=db mi-api-imagen

docker exec api ping db   # funciona: resolucion DNS por nombre en la red custom
```

## Como se usa en un proyecto real

1. Se crea una red custom por proyecto o por grupo de servicios relacionados, en vez de usar la red default.
2. Se configuran las aplicaciones para conectarse a otros servicios usando el nombre del container/servicio como hostname (`DB_HOST=db`), nunca una IP hardcodeada.
3. Se aislan grupos de servicios no relacionados en redes distintas, de forma que un container comprometido en una red no pueda alcanzar directamente servicios de otra red sin necesidad.
4. Se documenta el nombre de la red y los servicios conectados a ella como parte de la arquitectura del proyecto.
5. En Docker Compose, se aprovecha que esto ya viene resuelto automaticamente, sin necesitar crear la red manualmente.

## Criterio junior

Un junior debe poder crear una red custom y conectar dos containers que se comuniquen entre si por nombre.

## Criterio mid

Un mid debe diseñar la topologia de redes de un proyecto con varios servicios, decidiendo que servicios comparten red y cuales deben aislarse.

## Criterio senior

Un senior debe diseñar la segmentacion de red de todo el sistema como parte de la estrategia de seguridad (ver tambien capitulo de Seguridad), limitando que servicios pueden comunicarse entre si segun el principio de menor privilegio de red.

## Errores comunes

- Seguir usando IPs hardcodeadas incluso disponiendo de DNS interno por nombre, perdiendo la principal ventaja de las redes custom.
- Conectar todos los servicios de una organizacion a una unica red gigante, sin segmentacion, aumentando el riesgo si un servicio se ve comprometido.
- No documentar la topologia de redes de un proyecto con multiples servicios, dificultando el onboarding y el diagnostico de problemas de conectividad.
- Olvidar que containers en redes custom distintas no pueden comunicarse entre si por defecto, a menos que se conecten explicitamente a ambas.

## Practica

- [ ] Crea una red custom y conecta dos containers que se comuniquen por nombre.
- [ ] Verifica que un tercer container en una red distinta no puede alcanzar a los anteriores.
- [ ] Diseña la topologia de redes para un proyecto con al menos tres servicios relacionados.
- [ ] Conecta un container a dos redes distintas y verifica que puede comunicarse con ambos grupos.
- [ ] Documenta la topologia de red resultante.

## Preguntas de repaso

- Que ventaja concreta da una red custom sobre la red bridge default?
- Como resuelve Docker el nombre de un container a su IP dentro de una red custom?
- Por que segmentar servicios en redes distintas mejora la postura de seguridad de un sistema?
- Que pasa si dos containers estan en redes custom distintas y nunca se conectan explicitamente a una red compartida?
