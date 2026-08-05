---
tags:
  - Docker
  - DevOps
  - Docker-Imagenes
Creado: 2026-07-28
Relacionado:
  - Imagenes y Dockerfile
  - DOCKER
---

# 01 - Anatomia de una Imagen y Capas

## Que vas a aprender

En este capitulo vas a entender como esta construida una **imagen Docker** por dentro: el sistema de **capas** (union filesystem), como se comparten entre imagenes, y por que esto es clave para entender cache de build y tamaño de imagen.

## Explicacion

Una imagen Docker no es un unico bloque de datos, sino una pila de **capas** de solo lectura, cada una representando el resultado de una instruccion del Dockerfile (`RUN`, `COPY`, `ADD`). Estas capas se combinan mediante un **union filesystem** (overlay2 es el mas comun hoy) que las presenta como un unico sistema de archivos coherente. Cuando corre un container, Docker agrega una capa adicional de lectura-escritura encima de esas capas de solo lectura de la imagen.

La clave practica de esto es que las capas se **cachean e identifican por hash de contenido**, y pueden **compartirse entre imagenes distintas**: si dos imagenes usan la misma capa base (por ejemplo, la misma version de `node:20-alpine`), Docker no la descarga ni almacena dos veces. Esto tiene implicancias directas en velocidad de build (ver siguiente tema de cache) y en el tamaño real en disco de un conjunto de imagenes relacionadas.

## Modelo mental

Pensa una imagen como una torta de capas transparentes apiladas: cada capa agrega o modifica algo sobre las anteriores, pero las capas de abajo nunca cambian una vez creadas. Si dos tortas comparten las mismas capas de abajo, esas capas fisicamente existen una sola vez en el almacen de Docker, aunque las veas en dos tortas distintas.

## Ejemplo base

```bash
docker history mi-imagen:latest
# Muestra cada capa, su tamaño y el comando que la genero

docker inspect mi-imagen:latest --format '{{.RootFS.Layers}}'
# Lista los hashes de cada capa de la imagen
```

## Como se usa en un proyecto real

1. Se ordenan las instrucciones del Dockerfile de menos a mas cambiantes, para maximizar el reuso de capas cacheadas entre builds (ver siguiente tema).
2. Se usan imagenes base compartidas entre distintos servicios del equipo cuando es posible, para aprovechar el compartir capas y ahorrar espacio en disco de los servidores.
3. Se revisa `docker history` para identificar que instruccion agrega mas peso a una imagen antes de optimizarla.
4. Se entiende que borrar un archivo en una capa posterior no reduce el tamaño de la imagen si ese archivo fue creado en una capa anterior; el archivo sigue ocupando espacio en la capa donde se creo.
5. Se documenta la imagen base elegida y su justificacion, dado que define buena parte del tamaño y superficie de seguridad final.

## Criterio junior

Un junior debe entender que una imagen esta compuesta por capas apiladas, y poder inspeccionar esas capas con `docker history`.

## Criterio mid

Un mid debe identificar que capas de una imagen ocupan mas espacio y proponer reordenamientos o cambios de Dockerfile para reducirlas.

## Criterio senior

Un senior debe diseñar una estrategia de imagenes base compartidas para todo el equipo, entendiendo el impacto real en almacenamiento y velocidad de despliegue de compartir capas entre servicios.

## Errores comunes

- Asumir que borrar un archivo en una instruccion posterior del Dockerfile elimina el espacio que ocupaba en una capa anterior.
- No revisar `docker history` antes de intentar optimizar una imagen, optimizando a ciegas sin datos.
- Usar imagenes base distintas e innecesariamente variadas entre servicios del mismo equipo, perdiendo la oportunidad de compartir capas.
- No entender la relacion entre capas y cache de build, lo cual se profundiza en el siguiente tema.

## Practica

- [ ] Construye una imagen simple y revisa sus capas con `docker history`.
- [ ] Identifica la capa mas pesada de esa imagen.
- [ ] Verifica con `docker inspect` los hashes de capas de dos imagenes que compartan la misma base.
- [ ] Prueba borrar un archivo grande en una instruccion `RUN` posterior a la que lo creo, y observa que el tamaño de la imagen no baja como se esperaria.
- [ ] Anota una imagen base estandar que propondrias para tu equipo.

## Preguntas de repaso

- Que es una capa de imagen y como se relaciona con las instrucciones del Dockerfile?
- Por que borrar un archivo en una capa posterior no reduce el tamaño de una capa anterior?
- Que ventaja practica da que dos imagenes compartan las mismas capas base?
- Que herramienta usarias para inspeccionar el peso de cada capa de una imagen?
