---
tags:
  - Docker
  - DevOps
  - Docker-Stacks
Creado: 2026-07-28
Relacionado:
  - Docker para Diferentes Stacks
  - DOCKER
---

# 03 - Dockerizar un Frontend Angular

## Que vas a aprender

En este capitulo vas a aprender el patron especifico para dockerizar un **frontend Angular**: multi-stage build donde la etapa final no es Node en absoluto, sino un servidor web estatico (Nginx), sirviendo los archivos ya compilados.

## Explicacion

A diferencia de una API backend, una aplicacion Angular compilada para produccion (`ng build`) es un conjunto de archivos **estaticos** (HTML, CSS, JS) que no necesitan Node.js para ejecutarse, solo un servidor web que los sirva. Por eso el patron de multi-stage build aca es distinto al de una API: la primera etapa usa una imagen de Node para instalar dependencias y correr `ng build`, pero la etapa final usa una imagen de **Nginx**, copiando solo los archivos estaticos generados (`dist/`) hacia el directorio que Nginx sirve.

Un detalle importante para aplicaciones Angular con routing del lado del cliente (Angular Router) es configurar Nginx para redirigir cualquier ruta no encontrada hacia `index.html`, dado que el ruteo real lo maneja JavaScript en el navegador, no el servidor.

## Modelo mental

Pensa el build de Angular como imprimir un folleto completo (HTML, CSS, JS ya compilados y optimizados): una vez impreso, ya no necesitas la imprenta (Node.js) para mostrarselo a alguien, solo necesitas alguien que lo entregue (un servidor web simple como Nginx).

## Ejemplo base

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --configuration production

FROM nginx:alpine
COPY --from=build /app/dist/mi-app/browser /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

```nginx
# nginx.conf: soporte para Angular Router (client-side routing)
server {
    listen 80;
    root /usr/share/nginx/html;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

## Como se usa en un proyecto real

1. Se usa siempre Nginx (u otro servidor estatico liviano) como imagen final, nunca Node corriendo un servidor de desarrollo en produccion.
2. Se configura `try_files` en Nginx para soportar correctamente el routing del lado del cliente de Angular.
3. Se genera la imagen final extremadamente liviana (unos pocos MB con `nginx:alpine`), dado que solo contiene archivos estaticos y el servidor web.
4. Se configuran headers de cache apropiados en Nginx para los archivos estaticos con hash en el nombre (generados por Angular CLI), permitiendo cache agresivo del lado del cliente sin riesgo de servir version vieja.
5. Se conecta esta imagen con el backend (NestJS o .NET) via el mismo patron de redes visto en el capitulo correspondiente, tipicamente detras de un reverse proxy comun.

## Criterio junior

Un junior debe poder dockerizar una aplicacion Angular basica con multi-stage build usando Node para el build y Nginx para servir el resultado.

## Criterio mid

Un mid debe configurar Nginx correctamente para soportar el routing del lado del cliente de Angular sin errores 404 en rutas profundas.

## Criterio senior

Un senior debe optimizar headers de cache para archivos estaticos, y diseñar como este container se integra con el resto de la arquitectura (backend, reverse proxy) en produccion.

## Errores comunes

- Usar una imagen final con Node corriendo `ng serve` (servidor de desarrollo) en produccion, en vez de compilar y servir archivos estaticos con Nginx.
- No configurar `try_files` en Nginx, causando errores 404 al recargar la pagina en una ruta profunda del routing de Angular.
- No aprovechar headers de cache agresivos para archivos estaticos con hash en el nombre, perdiendo performance de carga en visitas repetidas.
- Copiar toda la carpeta `dist/` de forma generica sin verificar la ruta exacta que genera la version especifica de Angular usada (puede variar entre versiones).

## Practica

- [ ] Dockeriza una aplicacion Angular real con multi-stage build usando Node y Nginx.
- [ ] Configura `try_files` en Nginx y verifica que el routing del lado del cliente funciona al recargar una ruta profunda.
- [ ] Configura headers de cache apropiados para los archivos estaticos con hash.
- [ ] Mide el tamaño final de la imagen (deberia ser muy chico, solo Nginx + archivos estaticos).
- [ ] Conecta esta imagen con un backend en la misma red de Compose, detras de un reverse proxy comun.

## Preguntas de repaso

- Por que la imagen final de una aplicacion Angular no necesita Node.js en absoluto?
- Que problema resuelve `try_files $uri $uri/ /index.html` en la configuracion de Nginx?
- Que ventaja de cache tienen los archivos estaticos generados por Angular CLI con hash en el nombre?
- Como se integraria esta imagen con un backend y un reverse proxy en una arquitectura completa?
