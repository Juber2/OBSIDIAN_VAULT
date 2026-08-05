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

# 03 - Host y None Network

## Que vas a aprender

En este capitulo vas a conocer los modos de red **host** y **none**, alternativas menos comunes a bridge pero utiles en casos especificos: maxima performance de red y aislamiento total de red, respectivamente.

## Explicacion

El modo **host** (`--network host`) elimina el aislamiento de red entre el container y el host: el container comparte directamente la pila de red del host, sin NAT ni mapeo de puertos. Esto da la mejor performance de red posible (sin la sobrecarga de traduccion de direcciones), pero sacrifica el aislamiento y hace que el container ocupe puertos directamente en el host, con riesgo de colisiones si otro proceso ya usa ese puerto.

El modo **none** desconecta completamente al container de cualquier red: no tiene interfaz de red mas alla del loopback local. Es util para procesos batch que no necesitan ninguna conectividad de red (procesamiento local de archivos, calculos aislados), donde eliminar la red por completo reduce la superficie de ataque.

## Modelo mental

Pensa el modo host como quitarle las paredes al departamento del container: comparte directamente la calle (la red) con el resto del edificio (el host), gana velocidad de acceso pero pierde toda privacidad. El modo none es el opuesto: un departamento completamente sellado, sin puerta a la calle en absoluto.

## Ejemplo base

```bash
# Modo host: sin mapeo de puertos, el container usa directamente los puertos del host
docker run -d --network host nginx
# nginx escucha directamente en el puerto 80 del host, sin necesitar -p

# Modo none: sin ninguna interfaz de red
docker run --network none alpine ip addr
# Solo muestra la interfaz loopback, sin conectividad externa
```

## Como se usa en un proyecto real

1. Se usa el modo host solo cuando la performance de red es verdaderamente critica y se acepta el tradeoff de perder aislamiento (por ejemplo, ciertos casos de alto throughput de red).
2. Se evita el modo host por defecto en produccion multi-tenant, dado que rompe el aislamiento de red que Docker normalmente provee.
3. Se usa el modo none para procesos batch aislados que procesan datos locales sin necesitar ninguna conectividad, reduciendo la superficie de ataque al minimo posible.
4. Se documenta explicitamente por que un servicio usa modo host o none, dado que son excepciones al patron estandar de redes bridge custom.
5. Se es consciente de que el modo host, en Docker Desktop (Windows/Mac), tiene un comportamiento distinto y mas limitado que en Linux nativo.

## Criterio junior

Un junior debe entender que existen modos de red alternativos a bridge, y que el modo host elimina el aislamiento de red mientras que none lo maximiza.

## Criterio mid

Un mid debe decidir cuando el modo host se justifica por necesidades reales de performance, y usar el modo none para procesos que no necesitan ninguna conectividad.

## Criterio senior

Un senior debe evaluar criticamente cualquier uso de modo host en produccion como una excepcion documentada y justificada, dado el tradeoff de seguridad que implica, y establecer el criterio del equipo para cuando estos modos son aceptables.

## Errores comunes

- Usar modo host por comodidad (evitar mapear puertos) sin necesitar realmente la performance adicional, sacrificando aislamiento sin beneficio real.
- No anticipar colisiones de puertos al usar modo host, donde el container compite directamente por los mismos puertos que otros procesos del host.
- Asumir que el modo host se comporta igual en Docker Desktop que en Linux nativo, cuando en la practica tiene limitaciones distintas.
- Usar modo none donde en realidad se necesitaba algo de conectividad, causando fallos confusos de red.

## Practica

- [ ] Corre un container en modo host y verifica que usa directamente un puerto del host sin mapeo.
- [ ] Corre un container en modo none y confirma que no tiene conectividad externa.
- [ ] Investiga un caso real donde el modo host se justificaria por performance.
- [ ] Identifica un proceso batch de tu conocimiento que podria beneficiarse de modo none por seguridad.
- [ ] Documenta el criterio de tu equipo para aprobar el uso de modo host en produccion.

## Preguntas de repaso

- Que tradeoff especifico acepta el modo host a cambio de mejor performance de red?
- Por que el modo none reduce la superficie de ataque de un container?
- Que riesgo de colision de puertos introduce el modo host que bridge no tiene?
- En que caso real justificarias usar modo host en produccion?
