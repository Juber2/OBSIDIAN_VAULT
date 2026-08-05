---
tags:
  - Docker
  - DevOps
  - Docker-Volumenes
Creado: 2026-07-28
Relacionado:
  - Volumenes y Persistencia
  - DOCKER
---

# 02 - Gestion de Volumenes

## Que vas a aprender

En este capitulo vas a aprender a administrar volumenes en el dia a dia: crear, inspeccionar, listar, y limpiar volumenes sin usar (`prune`), evitando tanto la perdida accidental de datos como la acumulacion de basura en disco.

## Explicacion

Los comandos de gestion de volumenes (`docker volume create/ls/inspect/rm/prune`) permiten administrar el ciclo de vida de estos almacenes independientemente de los containers que los usan. Un volume puede existir sin ningun container usandolo activamente (por ejemplo, entre el momento en que se detiene un container y se crea el siguiente que lo reutiliza), lo cual es intencional: el volume no se elimina automaticamente al eliminar un container, a menos que se use `docker rm -v` explicitamente.

`docker volume prune` elimina todos los volumenes **no usados por ningun container** (ni corriendo ni detenido), lo cual es una operacion potencialmente destructiva si algun volume "huerfano" en realidad contenia datos importantes que se pensaban usar despues.

## Modelo mental

Pensa un volume como una caja de guardado independiente del inquilino (container) que la usa: cuando el inquilino se muda (el container se elimina), la caja se queda en el deposito hasta que alguien decida explicitamente tirarla. `prune` es literalmente "tirar todas las cajas que ningun inquilino actual esta usando", lo cual hay que hacer con cuidado de no tirar algo que todavia se necesitaba.

## Ejemplo base

```bash
docker volume create backups
docker volume ls
docker volume inspect backups
# Muestra la ubicacion real en el host, driver, y metadata

docker run --rm -v backups:/data alpine ls /data
# Inspecciona el contenido de un volume sin necesitar el container original

docker volume prune
# Elimina TODOS los volumenes no referenciados por ningun container (pide confirmacion)

docker rm -v mi-container
# Elimina el container Y sus volumenes anonimos asociados (los nombrados NO se borran)
```

## Como se usa en un proyecto real

1. Se nombran los volumenes explicitamente (`docker volume create nombre` o `-v nombre:/ruta`) en vez de dejar que Docker genere nombres anonimos dificiles de identificar despues.
2. Se revisa `docker volume ls` periodicamente para detectar volumenes huerfanos antes de decidir si limpiarlos.
3. Se usa `docker run --rm -v volumen:/data alpine ...` como patron para inspeccionar o manipular el contenido de un volume sin necesitar el container original de la aplicacion.
4. Se evita `docker volume prune` en servidores de produccion sin antes verificar manualmente que ningun volumen listado contiene datos criticos aun no respaldados.
5. Se documenta que volumenes de un proyecto son gestionados por Docker Compose (que los nombra con un prefijo del proyecto) para facilitar su identificacion.

## Criterio junior

Un junior debe poder crear, listar e inspeccionar volumenes basicos, y entender que eliminar un container no elimina automaticamente sus volumenes nombrados.

## Criterio mid

Un mid debe identificar y limpiar volumenes huerfanos de forma segura, verificando su contenido antes de eliminarlos con `prune`.

## Criterio senior

Un senior debe definir la politica de gestion de volumenes en produccion (nomenclatura, cuando limpiar, quien tiene permiso de ejecutar `prune`), evitando que una limpieza automatizada elimine datos criticos por error.

## Errores comunes

- Correr `docker volume prune` en un servidor de produccion sin revisar antes que volumenes serian eliminados.
- Depender de volumenes anonimos (sin nombre) que se vuelven dificiles de identificar y gestionar con el tiempo.
- Asumir que eliminar un container (`docker rm`) tambien elimina sus datos persistidos, sin verificar si el volume era nombrado (que sobrevive) o anonimo (que puede eliminarse con `-v`).
- No inspeccionar el contenido de un volume antes de decidir si es seguro eliminarlo.

## Practica

- [ ] Crea un volume nombrado y verifica su ubicacion real con `docker volume inspect`.
- [ ] Inspecciona el contenido de un volume usando un container temporal (`--rm`).
- [ ] Simula un volumen huerfano (elimina el container que lo usaba) y decide si limpiarlo o no.
- [ ] Investiga como Docker Compose nombra automaticamente los volumenes de un proyecto.
- [ ] Documenta la politica de limpieza de volumenes que aplicarias en un servidor de produccion.

## Preguntas de repaso

- Que diferencia hay entre un volume nombrado y uno anonimo respecto a su ciclo de vida?
- Que hace exactamente `docker volume prune` y por que es potencialmente riesgoso en produccion?
- Como inspeccionarias el contenido de un volume sin el container original que lo usaba?
- Por que conviene nombrar explicitamente los volumenes en vez de dejarlos anonimos?
