---
tags:
  - Docker
  - DevOps
  - Docker-Compose
Creado: 2026-07-28
Relacionado:
  - Docker Compose
  - DOCKER
---

# 02 - Servicios Redes y Volumenes en Compose

## Que vas a aprender

En este capitulo vas a profundizar en la configuracion de **servicios**, **redes** y **volumenes** dentro de Compose: dependencias entre servicios, redes custom explicitas cuando la red default de Compose no alcanza, y volumenes nombrados vs anonimos dentro del archivo.

## Explicacion

Ademas de la red automatica que Compose crea para el proyecto, se pueden definir **redes explicitas** en la seccion `networks:` cuando se necesita mas control: por ejemplo, separar una red "publica" (donde esta el reverse proxy) de una red "interna" (donde estan la base de datos y servicios que no deben ser alcanzables directamente). Un servicio puede conectarse a multiples redes declarando la lista en su propia seccion `networks:`.

Los volumenes se pueden declarar como **named** en la seccion `volumes:` de nivel superior (persisten y son referenciables por nombre entre corridas), o montarse directamente con una ruta de bind mount sin declaracion previa. La eleccion entre ambos sigue el mismo criterio visto en el capitulo de Volumenes y Persistencia, ahora expresado en YAML.

## Modelo mental

Pensa la seccion `networks:` de Compose como poder dibujar el plano de calles internas del edificio con mas detalle: no todos los departamentos necesitan acceso a todas las calles, y un buen plano de seguridad conecta cada departamento solo a las calles que realmente necesita.

## Ejemplo base

```yaml
services:
  proxy:
    image: nginx
    networks: [publica]
    ports: ["80:80"]

  api:
    build: .
    networks: [publica, interna]
    depends_on: [db]

  db:
    image: postgres:16
    networks: [interna]   # no accesible desde la red publica
    volumes:
      - datos-db:/var/lib/postgresql/data

networks:
  publica:
  interna:

volumes:
  datos-db:
```

## Como se usa en un proyecto real

1. Se separan redes publicas (donde esta el proxy expuesto) de redes internas (donde estan servicios que no deben exponerse directamente) usando la seccion `networks:` explicita.
2. Solo los servicios que realmente necesitan comunicarse entre redes distintas se conectan a mas de una.
3. Los volumenes de datos criticos siempre se declaran como named volumes en la seccion `volumes:` de nivel superior, no como bind mounts en produccion.
4. Se documenta en el propio `docker-compose.yml`, con comentarios breves, la razon de segmentaciones de red no obvias.
5. Se revisa periodicamente que ningun servicio sensible quedo accidentalmente conectado a una red mas amplia de lo necesario.

## Criterio junior

Un junior debe poder declarar redes y volumenes explicitos en un `docker-compose.yml` y conectar un servicio a mas de una red.

## Criterio mid

Un mid debe diseñar una segmentacion de redes basica (publica/interna) para un proyecto con un reverse proxy y servicios internos.

## Criterio senior

Un senior debe diseñar la topologia completa de redes y volumenes de un sistema con Compose como documento vivo de arquitectura, revisando en code review cualquier cambio que amplie innecesariamente el acceso de red de un servicio.

## Errores comunes

- Conectar todos los servicios a la misma red sin ninguna segmentacion, perdiendo la oportunidad de aislar servicios sensibles.
- No declarar volumenes named para datos criticos, dejandolos como bind mounts fragiles o, peor, sin persistencia en absoluto.
- Olvidar que un servicio necesita estar en dos redes cuando debe comunicarse con grupos distintos de servicios, causando fallos de conectividad confusos.
- No revisar la topologia de red periodicamente a medida que se agregan nuevos servicios al archivo.

## Practica

- [ ] Diseña un `docker-compose.yml` con al menos dos redes explicitas (publica e interna).
- [ ] Conecta un servicio a ambas redes y verifica que puede comunicarse con servicios de cada una.
- [ ] Verifica que un servicio en la red interna no es alcanzable directamente desde afuera.
- [ ] Declara un volumen named para datos criticos de ese proyecto.
- [ ] Documenta con comentarios en el YAML la razon de la segmentacion de redes elegida.

## Preguntas de repaso

- Por que separar servicios en redes publica e interna mejora la seguridad de la arquitectura?
- Como se conecta un servicio a mas de una red en Compose?
- Que diferencia hay entre declarar un volumen named en la seccion `volumes:` y usar un bind mount directo?
- Que revisarias en code review ante un cambio que amplia el acceso de red de un servicio sensible?
