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

# 03 - DNS y Resolucion de Nombres

## Que vas a aprender

En este capitulo vas a entender como Linux resuelve nombres de dominio a direcciones IP: el archivo `/etc/hosts`, la configuracion de resolvers DNS, y las herramientas de diagnostico `dig`/`nslookup`.

## Explicacion

Antes de consultar un servidor DNS externo, Linux revisa **`/etc/hosts`**, un archivo simple que mapea nombres a IPs manualmente, util para desarrollo local (apuntar un dominio a `127.0.0.1`) o para resolver nombres internos sin depender de un servidor DNS. Si el nombre no esta ahi, el sistema consulta los servidores DNS configurados en **`/etc/resolv.conf`** (aunque en sistemas modernos esto suele gestionarse automaticamente por `systemd-resolved` o NetworkManager, en vez de editarse directamente).

**`dig`** es la herramienta estandar para diagnosticar problemas de DNS en profundidad, mostrando exactamente que respuesta da un servidor DNS especifico para un dominio, incluyendo tiempos de respuesta y el tipo de registro consultado (A, AAAA, CNAME, MX). Esto conecta directamente con el capitulo de Redes del libro de Docker: el DNS interno que Docker provee en redes custom es, conceptualmente, el mismo mecanismo aplicado dentro de una red de containers.

## Modelo mental

Pensa `/etc/hosts` como una agenda personal de contactos que consultas primero antes de buscar en la guia telefonica publica (DNS): si el numero que buscas ya esta en tu agenda personal, ni siquiera necesitas consultar la guia externa. `dig` es como llamar directamente a la central telefonica (un servidor DNS especifico) y pedirle explicitamente que numero tiene registrado para un nombre, sin pasar por ningun atajo.

## Ejemplo base

```bash
cat /etc/hosts
# 127.0.0.1   localhost
# 192.168.1.50  servidor-interno.local

dig google.com                     # consulta DNS completa, con tiempos y detalles
dig +short google.com               # solo la IP resultante, sin detalles adicionales
dig @8.8.8.8 google.com             # consulta especificamente a un servidor DNS (Google) 

nslookup google.com                 # alternativa mas simple a dig
```

## Como se usa en un proyecto real

1. Se agrega una entrada en `/etc/hosts` para desarrollo local, apuntando un dominio de prueba a `127.0.0.1` sin necesitar configurar DNS real.
2. Se usa `dig` para diagnosticar problemas donde un dominio "no resuelve" o resuelve a una IP inesperada, verificando exactamente que responde cada servidor DNS involucrado.
3. Se entiende que `/etc/hosts` tiene prioridad sobre DNS, explicando comportamientos donde un dominio resuelve distinto de lo esperado por una entrada manual olvidada.
4. Se conecta este conocimiento con el DNS interno de Docker (capitulo de Redes de ese libro), entendiendo el mismo principio de resolucion de nombres aplicado a escala de una red de containers.
5. Se verifica la configuracion de resolvers DNS del sistema al diagnosticar problemas de conectividad que resultan ser, en realidad, problemas de resolucion de nombres.

## Criterio junior

Un junior debe poder agregar una entrada en `/etc/hosts` y verificar una resolucion DNS basica con `dig` o `nslookup`.

## Criterio mid

Un mid debe diagnosticar problemas de resolucion DNS comparando lo que dice `/etc/hosts` contra lo que responde un servidor DNS especifico con `dig`.

## Criterio senior

Un senior debe diseñar la estrategia de resolucion de nombres de infraestructura compleja, coordinando DNS interno de containers/orquestadores con DNS externo publico, y diagnosticar problemas de propagacion o cache de DNS en sistemas distribuidos.

## Errores comunes

- Olvidar una entrada de desarrollo en `/etc/hosts`, causando comportamiento inesperado meses despues cuando ese dominio deberia resolver de forma distinta.
- No usar `dig` para diagnosticar un problema de DNS, quedandose solo con herramientas mas basicas que dan menos informacion util.
- No entender que `/etc/hosts` tiene prioridad sobre DNS externo, causando confusion al depurar por que un dominio no resuelve como se espera.
- Confundir un problema de conectividad de red con un problema de resolucion DNS, perdiendo tiempo diagnosticando en el lugar equivocado.

## Practica

- [ ] Agrega una entrada en `/etc/hosts` para un dominio de prueba apuntando a `127.0.0.1`.
- [ ] Usa `dig` para consultar la resolucion real de un dominio publico.
- [ ] Compara la respuesta de `dig @8.8.8.8` contra otro servidor DNS para el mismo dominio.
- [ ] Diagnostica (simulado) un caso donde `/etc/hosts` causa comportamiento inesperado.
- [ ] Anota como conecta este conocimiento con el DNS interno de redes Docker visto en el libro correspondiente.

## Preguntas de repaso

- Que archivo consulta Linux antes de recurrir a un servidor DNS externo?
- Que informacion adicional da `dig` sobre `nslookup` al diagnosticar un problema de DNS?
- Por que una entrada olvidada en `/etc/hosts` puede causar comportamiento confuso meses despues?
- Como se relaciona este conocimiento con el DNS interno de las redes custom de Docker?
