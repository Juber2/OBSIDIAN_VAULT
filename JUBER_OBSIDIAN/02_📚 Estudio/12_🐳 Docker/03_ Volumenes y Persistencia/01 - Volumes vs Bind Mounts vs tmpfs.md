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

# 01 - Volumes vs Bind Mounts vs tmpfs

## Que vas a aprender

En este capitulo vas a entender las tres formas de manejar datos que sobreviven (o no) a un container en Docker: **volumes**, **bind mounts**, y **tmpfs mounts**, y cuando usar cada uno.

## Explicacion

Por defecto, cualquier dato escrito dentro de un container vive en su capa de lectura-escritura, que **desaparece cuando el container se elimina**. Para persistir o compartir datos, Docker ofrece tres mecanismos: un **volume** es un almacenamiento gestionado completamente por Docker, viviendo en una ubicacion del host que Docker administra, ideal para datos de aplicacion (bases de datos, uploads). Un **bind mount** monta una ruta especifica del filesystem del host directamente dentro del container, util en desarrollo para reflejar cambios de codigo en vivo. Un **tmpfs mount** vive solo en memoria RAM del host, nunca toca disco, y desaparece al detener el container, util para datos temporales sensibles que no deben persistir.

## Modelo mental

Pensa un volume como una caja de guardado que Docker administra por vos, sin que necesites saber exactamente donde esta fisicamente. Un bind mount es como abrir una ventana directa a una carpeta especifica de tu propia casa (el host) desde adentro del container. Un tmpfs es una pizarra que se borra sola apenas apagas la luz (RAM, sin persistencia en disco).

## Ejemplo base

```bash
# Volume gestionado por Docker
docker volume create datos-db
docker run -d -v datos-db:/var/lib/postgresql/data postgres:16

# Bind mount para desarrollo (refleja cambios de codigo en vivo)
docker run -d -v $(pwd)/src:/app/src mi-imagen-dev

# tmpfs para datos temporales en memoria, nunca en disco
docker run -d --tmpfs /app/cache:size=100m mi-imagen
```

## Como se usa en un proyecto real

1. Se usan **volumes** para datos de aplicacion que deben persistir independientemente del ciclo de vida del container (bases de datos, archivos subidos por usuarios).
2. Se usan **bind mounts** principalmente en desarrollo local, para que cambios de codigo en el host se reflejen sin reconstruir la imagen.
3. Se evitan bind mounts en produccion para el codigo de la aplicacion; en produccion la imagen ya deberia contener el codigo final (inmutabilidad, ver capitulo Senior).
4. Se usa **tmpfs** para datos sensibles de corta vida (por ejemplo, archivos temporales de procesamiento) que nunca deberian quedar escritos en el disco del host.
5. Se documenta que volumes son criticos de respaldar (ver siguiente tema de backup) y cuales son descartables sin riesgo.

## Criterio junior

Un junior debe entender que sin un volume o bind mount, los datos de un container se pierden al eliminarlo, y poder montar un volume basico.

## Criterio mid

Un mid debe elegir correctamente entre volume, bind mount y tmpfs segun el caso de uso, y usar bind mounts efectivamente en un flujo de desarrollo local.

## Criterio senior

Un senior debe diseñar la estrategia de persistencia de datos de todo el sistema, decidiendo que vive en volumes gestionados, que se excluye de persistencia por seguridad (tmpfs), y garantizando que produccion nunca dependa de bind mounts de codigo fuente mutable.

## Errores comunes

- Perder datos importantes porque nunca se monto un volume, asumiendo erroneamente que los datos del container persistirian solos.
- Usar bind mounts de codigo fuente en produccion, rompiendo la inmutabilidad de la imagen desplegada.
- Guardar datos sensibles temporales en un volume normal cuando deberian vivir solo en memoria con tmpfs.
- No distinguir claramente, en la documentacion del proyecto, que volumes son criticos de backup y cuales son regenerables sin riesgo.

## Practica

- [ ] Crea un volume gestionado por Docker y monta una base de datos sobre el.
- [ ] Elimina el container de esa base de datos y verifica que los datos persisten al crear uno nuevo apuntando al mismo volume.
- [ ] Configura un bind mount para reflejar cambios de codigo en vivo en un entorno de desarrollo.
- [ ] Usa un tmpfs mount para un caso de datos temporales sensibles.
- [ ] Documenta que volumes de tu proyecto son criticos y cuales son descartables.

## Preguntas de repaso

- Que pasa con los datos de un container si no se monta ningun volume ni bind mount?
- Cuando conviene un bind mount sobre un volume gestionado por Docker?
- Por que un tmpfs mount es apropiado para datos sensibles de corta vida?
- Por que produccion no deberia depender de bind mounts de codigo fuente?
