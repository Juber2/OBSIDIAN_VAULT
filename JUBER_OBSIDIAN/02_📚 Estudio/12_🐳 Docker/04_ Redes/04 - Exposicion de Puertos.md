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

# 04 - Exposicion de Puertos

## Que vas a aprender

En este capitulo vas a dominar el mapeo de puertos con `-p`/`--publish`: la diferencia entre `EXPOSE` (documentacion) y `-p` (publicacion real), el formato completo de mapeo, y consideraciones de seguridad al exponer puertos.

## Explicacion

`EXPOSE` en el Dockerfile es puramente documentacion e informacion para otras herramientas; **no publica nada por si solo**. Lo que realmente hace que un puerto del container sea accesible desde el host es la opcion `-p host:container` en `docker run` (o el equivalente `ports:` en Compose), que crea una regla de NAT desde un puerto del host hacia el puerto interno del container.

El formato completo es `-p [IP-del-host:]puerto-host:puerto-container[/protocolo]`. Omitir la IP del host (`-p 3000:3000`) publica en todas las interfaces del host (`0.0.0.0`), lo cual en un servidor con IP publica significa exponer ese puerto a internet; especificar `-p 127.0.0.1:3000:3000` limita el acceso solo a conexiones locales del host, una practica de seguridad importante para servicios que no deben ser publicos (como una base de datos).

## Modelo mental

Pensa `EXPOSE` como un cartel informativo en la puerta del departamento diciendo "aca hay una puerta en el numero 3000", pero sin puerta real todavia. `-p` es literalmente instalar la puerta y decidir a que calle da: si no especificas una direccion (IP), la puerta queda abierta a cualquier calle que llegue hasta el edificio (todas las interfaces del host).

## Ejemplo base

```bash
docker run -d -p 3000:3000 mi-api
# Publica en TODAS las interfaces del host: accesible desde cualquier IP que llegue al servidor

docker run -d -p 127.0.0.1:5432:5432 postgres:16
# Publica solo en localhost del host: la base de datos NO es accesible desde afuera del servidor

docker run -d -p 8080:80/tcp -p 8080:80/udp mi-servicio
# Mapea el mismo puerto para ambos protocolos si el servicio los necesita
```

## Como se usa en un proyecto real

1. Se publican solo los puertos estrictamente necesarios; servicios internos (bases de datos, colas) que solo necesitan otros containers de la misma red **no** se publican al host en absoluto.
2. Cuando un puerto interno si debe ser accesible desde el host (por ejemplo, para debugging local), se limita a `127.0.0.1` en vez de todas las interfaces, especialmente en servidores con IP publica.
3. Se documenta claramente en el `docker-compose.yml` o en scripts de deploy que puertos se publican y por que, como parte de la superficie de exposicion del sistema.
4. Se usa un reverse proxy (Nginx, Traefik) como unico punto publicamente expuesto, mientras los servicios de aplicacion se comunican entre si por la red interna sin publicar puertos individuales al host.
5. Se revisan periodicamente los puertos publicados en servidores de produccion como parte de auditorias de seguridad.

## Criterio junior

Un junior debe entender la diferencia entre `EXPOSE` y `-p`, y poder publicar un puerto basico de forma correcta.

## Criterio mid

Un mid debe limitar la publicacion de puertos sensibles a `127.0.0.1`, y decidir que servicios de una arquitectura realmente necesitan un puerto publicado al host.

## Criterio senior

Un senior debe diseñar la superficie de exposicion de red de todo el sistema, usando un reverse proxy como unico punto de entrada publico y auditando regularmente que puertos estan efectivamente expuestos en los servidores de produccion.

## Errores comunes

- Publicar un puerto de base de datos o servicio interno sin necesidad, exponiendolo accidentalmente a internet en un servidor con IP publica.
- Confundir `EXPOSE` en el Dockerfile con publicacion real, asumiendo que un puerto "expuesto" ya es accesible desde el host.
- No limitar la IP de publicacion (`127.0.0.1`) para servicios que solo deben ser accesibles localmente en el servidor.
- No auditar periodicamente los puertos publicados en produccion, dejando exposiciones olvidadas de versiones anteriores del sistema.

## Practica

- [ ] Publica un puerto de un container en todas las interfaces y verifica accesibilidad.
- [ ] Publica otro puerto limitado a `127.0.0.1` y verifica que no es accesible externamente.
- [ ] Revisa un `docker-compose.yml` existente e identifica puertos que no deberian estar publicados.
- [ ] Diseña una arquitectura donde solo un reverse proxy publica un puerto, y el resto de servicios se comunican internamente.
- [ ] Audita los puertos publicados en un servidor real (o de practica) que administres.

## Preguntas de repaso

- Que diferencia hay entre `EXPOSE` en el Dockerfile y `-p` en `docker run`?
- Que significa omitir la IP del host en `-p puerto:puerto`, y por que puede ser un riesgo de seguridad?
- Por que un servicio interno como una base de datos generalmente no deberia publicar su puerto al host?
- Que ventaja de seguridad da usar un reverse proxy como unico punto de entrada publico?
