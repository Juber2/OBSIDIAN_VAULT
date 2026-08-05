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

# 01 - Bridge Network por Defecto

## Que vas a aprender

En este capitulo vas a entender la red **bridge** por defecto de Docker: como los containers obtienen una IP interna, como se comunican entre si, y las limitaciones de la red bridge default que motivan usar redes custom (siguiente tema).

## Explicacion

Al instalar Docker, se crea automaticamente una red bridge llamada `bridge`, y cualquier container corrido sin especificar red se conecta a ella. Docker crea una interfaz de red virtual en el host y asigna a cada container una IP interna dentro de un rango privado, permitiendo que los containers se comuniquen entre si por IP dentro de esa red.

La limitacion clave de la red bridge **default** (a diferencia de redes bridge custom) es que **no provee resolucion DNS automatica por nombre de container**: dos containers en la red default solo pueden comunicarse por IP, lo cual es fragil porque las IPs pueden cambiar. Esta limitacion especifica es la razon principal por la que en la practica casi nunca se usa la red default para comunicacion entre containers, y en cambio se crean redes custom (ver siguiente tema).

## Modelo mental

Pensa la red bridge default como un edificio de departamentos donde cada inquilino tiene un numero de puerta (IP), pero no hay ningun cartel con nombres en las puertas: para visitar a alguien tenes que saber su numero exacto de memoria, y si se muda de puerta (cambia de IP), perdiste el rastro.

## Ejemplo base

```bash
docker run -d --name app1 alpine sleep 1000
docker run -d --name app2 alpine sleep 1000

docker inspect -f '{{.NetworkSettings.IPAddress}}' app1
# IP interna de app1, por ejemplo 172.17.0.2

docker exec app2 ping 172.17.0.2   # funciona: comunicacion por IP
docker exec app2 ping app1         # falla en la red default: no hay DNS por nombre
```

## Como se usa en un proyecto real

1. Se evita depender de la red `bridge` default para comunicacion entre containers de una misma aplicacion, precisamente por la falta de DNS por nombre.
2. Se usa la red default principalmente para containers aislados y sin necesidad de comunicarse con otros por nombre (casos puntuales de testing rapido).
3. Se investiga siempre, ante un problema de "un container no puede hablar con otro por nombre", si ambos estan en la misma red custom o si uno quedo en la red default por omision.
4. Se documenta que cualquier `docker run` sin `--network` especificado termina en la red default, como recordatorio para nuevos integrantes del equipo.
5. Se entiende esta limitacion como la motivacion directa para el siguiente tema: redes bridge custom con DNS automatico.

## Criterio junior

Un junior debe entender que existe una red bridge default, que cada container recibe una IP interna, y que esa red no resuelve nombres de container automaticamente.

## Criterio mid

Un mid debe diagnosticar rapidamente cuando un problema de comunicacion entre containers se debe a estar en la red default en vez de una red custom.

## Criterio senior

Un senior debe establecer como estandar del equipo nunca depender de la red bridge default para comunicacion entre servicios, documentando esta decision como parte de las buenas practicas de red del proyecto.

## Errores comunes

- Intentar que dos containers se comuniquen por nombre en la red default y frustrarse porque "Docker no resuelve nombres", sin saber que es una limitacion especifica de esa red en particular.
- Depender de IPs internas hardcodeadas, que pueden cambiar si los containers se recrean.
- No especificar `--network` en `docker run`, terminando sin darse cuenta en la red default para servicios que si necesitan comunicarse entre si.
- Confundir la red bridge default con las redes bridge custom, asumiendo que ambas se comportan igual.

## Practica

- [ ] Corre dos containers sin especificar red y verifica que terminan en la red `bridge` default.
- [ ] Intenta que se comuniquen por nombre y confirma que falla.
- [ ] Verifica que si pueden comunicarse por IP interna.
- [ ] Inspecciona la red default con `docker network inspect bridge`.
- [ ] Anota la regla que aplicarias en tu equipo sobre nunca depender de la red default para servicios relacionados.

## Preguntas de repaso

- Que limitacion especifica tiene la red bridge default respecto a la resolucion de nombres?
- Por que depender de IPs internas hardcodeadas es fragil?
- Que red usa un container si no se especifica ninguna con `--network`?
- Que motiva el uso de redes bridge custom en vez de la default?
