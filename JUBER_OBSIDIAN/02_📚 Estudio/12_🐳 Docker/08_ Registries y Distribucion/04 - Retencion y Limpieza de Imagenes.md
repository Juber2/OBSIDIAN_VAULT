---
tags:
  - Docker
  - DevOps
  - Docker-Registries
Creado: 2026-07-28
Relacionado:
  - Registries y Distribucion
  - DOCKER
---

# 04 - Retencion y Limpieza de Imagenes

## Que vas a aprender

En este capitulo vas a aprender a diseñar una **politica de retencion** de imagenes en un registry, evitando tanto la acumulacion indefinida de imagenes viejas (costo de almacenamiento) como el borrado accidental de una imagen que todavia esta desplegada en algun ambiente.

## Explicacion

Con una estrategia de tagging por commit (ver tema anterior), cada push a la rama principal genera una imagen nueva, y sin limpieza, un registry acumula miles de imagenes con el tiempo, generando costo de almacenamiento innecesario. La mayoria de registries (ECR, GCR, Docker Hub) ofrecen **politicas de ciclo de vida** configurables: por ejemplo, "mantener las ultimas 50 imagenes de cada rama" o "eliminar imagenes sin tag `release` con mas de 90 dias de antiguedad".

El riesgo a evitar es eliminar una imagen que, aunque parezca vieja, todavia esta corriendo en algun ambiente (un ambiente de staging que no se actualizo hace tiempo, o un rollback pendiente). Por eso las politicas de retencion inteligentes suelen excluir explicitamente tags marcados como "release" o "produccion actual" de cualquier limpieza automatica.

## Modelo mental

Pensa la retencion de imagenes como organizar un deposito: no podes guardar cada caja para siempre (el espacio y costo crecen sin limite), pero tampoco podes tirar cajas al azar sin revisar si alguna todavia esta en uso activo en algun lugar de la empresa. La solucion es un sistema de etiquetas claras que distinga "en uso, nunca tirar" de "candidato a limpieza automatica".

## Ejemplo base

```
Politica de retencion tipica (conceptual, la sintaxis exacta depende del registry):

- Tags que empiezan con "sha-" (builds de CI): retener solo los ultimos 30, o menos de 30 dias.
- Tags de version semantica (releases formales): retener indefinidamente, nunca limpiar automaticamente.
- Tags "latest" y "produccion-actual": excluidos siempre de cualquier limpieza.
- Imagenes sin ningun tag (dangling): limpiar automaticamente despues de 7 dias.
```

## Como se usa en un proyecto real

1. Se configura una politica de ciclo de vida en el registry que distinga claramente entre imagenes de build efimeras y releases formales que deben persistir.
2. Se excluyen explicitamente de cualquier limpieza automatica los tags que representan lo que esta corriendo actualmente en produccion o staging.
3. Se revisa periodicamente el costo de almacenamiento del registry como señal de si la politica de retencion esta funcionando correctamente.
4. Se documenta la politica de retencion vigente, para que el equipo entienda que puede y que no puede eliminarse manualmente sin riesgo.
5. Se limpian imagenes "dangling" (sin ningun tag, resultado de builds sobreescritos) de forma mas agresiva, dado que nunca deberian estar en uso real.

## Criterio junior

Un junior debe entender la diferencia entre una imagen de build efimera y un release formal, y por que no se limpian con la misma agresividad.

## Criterio mid

Un mid debe configurar una politica de ciclo de vida basica en un registry, excluyendo explicitamente tags criticos de cualquier limpieza automatica.

## Criterio senior

Un senior debe diseñar la politica de retencion completa de la organizacion, balanceando costo de almacenamiento contra el riesgo de eliminar accidentalmente algo en uso, y auditando periodicamente que la politica siga siendo correcta a medida que el sistema evoluciona.

## Errores comunes

- No tener ninguna politica de retencion, acumulando costo de almacenamiento indefinidamente sin control.
- Configurar una politica demasiado agresiva que elimina una imagen todavia desplegada en algun ambiente, causando un incidente al intentar hacer rollback.
- No excluir explicitamente tags de release o de produccion actual de las reglas de limpieza automatica.
- No revisar el costo real de almacenamiento del registry, perdiendo visibilidad de si la politica vigente es efectiva.

## Practica

- [ ] Diseña una politica de retencion para un registry de practica, distinguiendo builds efimeros de releases formales.
- [ ] Configura esa politica en un registry real (ECR, GCR, Docker Hub) si es posible.
- [ ] Verifica que los tags de produccion actual esten explicitamente excluidos de cualquier limpieza.
- [ ] Limpia imagenes "dangling" (sin tag) de forma manual como practica.
- [ ] Documenta la politica de retencion de tu proyecto para referencia del equipo.

## Preguntas de repaso

- Que riesgo especifico hay en una politica de retencion demasiado agresiva?
- Por que los tags de release formal deberian excluirse de la limpieza automatica basada en antiguedad?
- Que son las imagenes "dangling" y por que se pueden limpiar mas agresivamente que las tageadas?
- Como verificarias que la politica de retencion vigente sigue siendo apropiada a medida que el sistema crece?
