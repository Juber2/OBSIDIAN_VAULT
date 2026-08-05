---
tags:
  - Linux
  - Redes
  - Linux-Redes
Creado: 2026-07-28
Relacionado:
  - Redes
  - LINUX
---

# 01 - Fundamentos de Redes TCP IP en Linux

## Que vas a aprender

En este capitulo vas a repasar los fundamentos de redes aplicados especificamente a Linux: modelo TCP/IP, direcciones IP y subredes, puertos, y las herramientas basicas de diagnostico que ya viste aplicadas de forma practica en el libro de Docker.

## Explicacion

El modelo TCP/IP organiza la comunicacion de red en capas: la capa de **red** (IP) se encarga de rutear paquetes entre dispositivos usando direcciones IP; la capa de **transporte** (TCP/UDP) se encarga de la entrega confiable (TCP, con confirmacion y reintentos) o rapida sin garantias (UDP) de datos entre aplicaciones especificas, identificadas por **puertos**. Una direccion IP identifica un dispositivo en una red; una **mascara de subred** (o notacion CIDR como `/24`) define que porcion de esa direccion identifica la red y cual identifica el dispositivo especifico dentro de ella.

Estos son exactamente los mismos conceptos ya aplicados en el capitulo de Redes del libro de Docker (bridge networks, exposicion de puertos): un container conectado a una red Docker recibe una IP dentro de una subred privada, y publicar un puerto (`-p`) es literalmente mapear un puerto TCP/UDP del host hacia el container.

## Modelo mental

Pensa una direccion IP como la direccion postal completa de un edificio, y un puerto como el numero de departamento especifico dentro de ese edificio: el cartero (el paquete de red) usa la direccion para llegar al edificio correcto, y el numero de departamento para saber exactamente a quien entregarlo dentro de ese edificio.

## Ejemplo base

```bash
ip addr show                  # interfaces de red y sus direcciones IP actuales
ip route show                  # tabla de rutas: como el sistema llega a distintas redes

ping -c 4 8.8.8.8               # verifica conectividad basica a nivel IP
traceroute google.com           # muestra la ruta de saltos hasta un destino

ss -tulnp                       # puertos abiertos (TCP/UDP) y que proceso los usa
```

## Como se usa en un proyecto real

1. Se usa `ip addr` y `ip route` como primeras herramientas de diagnostico ante cualquier problema de conectividad en un servidor.
2. Se entiende la notacion CIDR (`/24`, `/16`) al configurar redes de infraestructura o revisar reglas de firewall que restringen acceso por rango de IPs.
3. Se distingue TCP de UDP al diseñar servicios: TCP para donde la confiabilidad importa (HTTP, bases de datos), UDP para donde la velocidad importa mas que garantizar cada paquete (DNS, streaming en tiempo real).
4. Se conecta este conocimiento directamente con el capitulo de Redes del libro de Docker, entendiendo que las redes de containers son una aplicacion practica de estos mismos fundamentos.
5. Se usa `ss` (sucesor moderno de `netstat`) para verificar que puertos estan realmente escuchando en un servidor, relevante al diagnosticar por que un servicio no es accesible.

## Criterio junior

Un junior debe poder identificar la direccion IP de un servidor con `ip addr`, y verificar conectividad basica con `ping`.

## Criterio mid

Un mid debe interpretar notacion CIDR de subredes, distinguir cuando usar TCP vs UDP para un servicio, y usar `ss` para verificar puertos en escucha.

## Criterio senior

Un senior debe diseñar la topologia de red de infraestructura completa aplicando estos fundamentos, coordinando con el capitulo de Redes del libro de Docker para entender como las redes de containers se integran con la red del host.

## Errores comunes

- No revisar `ip addr`/`ip route` como primer paso ante un problema de conectividad, perdiendo tiempo en diagnosticos mas complejos innecesariamente.
- Confundir cuando un servicio deberia usar TCP vs UDP, eligiendo el protocolo equivocado para el caso de uso real.
- No entender notacion CIDR, causando configuraciones de firewall o red mas amplias o mas restrictivas de lo pretendido.
- Usar `netstat` (obsoleto en muchas distribuciones modernas) en vez de `ss`, que es mas rapido y es el reemplazo estandar actual.

## Practica

- [ ] Revisa las interfaces de red y direcciones IP de tu sistema con `ip addr`.
- [ ] Revisa la tabla de rutas con `ip route` y entiende como el sistema llega a la red externa.
- [ ] Verifica conectividad con `ping` y traza la ruta con `traceroute` hacia un destino externo.
- [ ] Lista puertos en escucha en tu sistema con `ss -tulnp`.
- [ ] Anota que servicios de tu conocimiento usan TCP y cuales usan UDP, y por que.

## Preguntas de repaso

- Que diferencia hay entre una direccion IP y un puerto en terminos de lo que identifican?
- Que diferencia fundamental hay entre TCP y UDP?
- Que informacion util da `ip route` sobre como el sistema se conecta a redes externas?
- Como conecta este conocimiento con lo aprendido sobre redes de Docker?
