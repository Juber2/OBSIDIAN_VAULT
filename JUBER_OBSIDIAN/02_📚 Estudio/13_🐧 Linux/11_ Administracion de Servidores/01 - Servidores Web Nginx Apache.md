---
tags:
  - Linux
  - Administracion
  - Linux-Servidores
Creado: 2026-07-28
Relacionado:
  - Administracion de Servidores
  - LINUX
---

# 01 - Servidores Web Nginx Apache

## Que vas a aprender

En este capitulo vas a aprender a administrar **Nginx** (y comparar con Apache) como servidor web y **reverse proxy**, conectando directamente con lo ya visto en el capitulo de Docker para Diferentes Stacks (dockerizar un frontend Angular con Nginx).

## Explicacion

Nginx es conocido por su arquitectura eficiente basada en eventos (maneja muchas conexiones concurrentes con poco consumo de recursos), siendo la eleccion mas comun hoy tanto para servir archivos estaticos como para actuar de **reverse proxy**: recibir trafico publico y redirigirlo hacia servicios backend internos (una API NestJS, una app .NET), centralizando SSL, compresion, y balanceo de carga en un unico punto. Apache (con su modulo `mod_php` historico) sigue siendo comun en hosting compartido tradicional, con un modelo de procesos distinto y mayor flexibilidad de configuracion por directorio via `.htaccess`.

Como reverse proxy, Nginx recibe una request publica y la reenvia a un backend interno (`proxy_pass`), pudiendo agregar headers, terminar SSL/TLS antes de que el trafico llegue al backend (que entonces solo necesita hablar HTTP plano internamente), y servir como el unico punto de entrada expuesto, exactamente el patron de arquitectura ya visto en el capitulo de Redes del libro de Docker (solo el proxy publica un puerto, el resto de servicios quedan en la red interna).

## Modelo mental

Pensa Nginx como reverse proxy como la recepcion de un edificio de oficinas: los visitantes (requests publicas) llegan a un unico punto de entrada, la recepcion verifica y redirige a cada visitante hacia la oficina interna correcta (el backend correspondiente), sin que el visitante necesite saber en que piso o pasillo especifico esta esa oficina.

## Ejemplo base

```nginx
# /etc/nginx/sites-available/mi-app
server {
    listen 80;
    server_name miapp.com;

    location / {
        proxy_pass http://localhost:3000;   # reenvia a la API NestJS interna
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        root /var/www/miapp;    # sirve archivos estaticos directamente, sin pasar por el backend
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/mi-app /etc/nginx/sites-enabled/
sudo nginx -t                    # verifica la sintaxis de configuracion antes de aplicar
sudo systemctl reload nginx       # aplica cambios sin cortar conexiones activas
```

## Como se usa en un proyecto real

1. Se usa Nginx como unico punto de entrada publico, redirigiendo internamente a servicios backend que no publican sus puertos directamente, aplicando el mismo principio de arquitectura de red ya visto en Docker.
2. Se verifica siempre la sintaxis (`nginx -t`) antes de aplicar cambios de configuracion, evitando dejar el servidor sin servir trafico por un error tipografico.
3. Se usa `reload` en vez de `restart` para aplicar cambios de configuracion sin cortar conexiones activas existentes.
4. Se sirven archivos estaticos directamente desde Nginx (sin pasar por el backend de aplicacion), aprovechando su eficiencia para ese caso especifico.
5. Se conecta esto directamente con el ejemplo de Nginx sirviendo un frontend Angular ya visto en el libro de Docker, y con la terminacion de SSL vista en el capitulo Senior de ese mismo libro.

## Criterio junior

Un junior debe poder configurar un `server block` basico de Nginx para servir archivos estaticos o hacer proxy hacia un backend simple.

## Criterio mid

Un mid debe configurar Nginx como reverse proxy completo, con headers apropiados, y verificar/aplicar cambios de configuracion de forma segura.

## Criterio senior

Un senior debe diseñar la arquitectura completa de entrada de trafico de un sistema usando Nginx como reverse proxy central, coordinando con la arquitectura de red de containers vista en Docker.

## Errores comunes

- No verificar la sintaxis con `nginx -t` antes de aplicar cambios, arriesgando que el servidor deje de servir trafico por un error de configuracion.
- Usar `restart` en vez de `reload`, cortando conexiones activas innecesariamente para un cambio de configuracion simple.
- Exponer directamente el backend de aplicacion sin pasar por el reverse proxy, perdiendo el punto centralizado de control de SSL, headers, y balanceo.
- No servir archivos estaticos directamente desde Nginx cuando seria mas eficiente, haciendolos pasar innecesariamente por el backend de aplicacion.

## Practica

- [ ] Configura un `server block` de Nginx que sirva archivos estaticos.
- [ ] Configura otro que actue como reverse proxy hacia un backend local de practica.
- [ ] Verifica la sintaxis con `nginx -t` antes de aplicar, y aplica con `reload`.
- [ ] Compara conceptualmente Nginx y Apache en cuanto a arquitectura de manejo de conexiones.
- [ ] Conecta esta configuracion con el ejemplo de Nginx sirviendo Angular visto en el libro de Docker.

## Preguntas de repaso

- Que hace un reverse proxy y por que centralizar el punto de entrada publico es una buena practica arquitectonica?
- Por que conviene usar `reload` en vez de `restart` al aplicar cambios de configuracion?
- Que verificarias siempre antes de aplicar un cambio de configuracion de Nginx en produccion?
- Como se conecta esta configuracion con lo ya visto sobre dockerizar un frontend Angular con Nginx?
