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

# 01 - Docker Hub vs Registries Privados

## Que vas a aprender

En este capitulo vas a comparar **Docker Hub** con **registries privados** (ECR de AWS, GCR/Artifact Registry de Google, Harbor auto-hospedado), y los criterios para elegir donde alojar las imagenes de un proyecto real.

## Explicacion

**Docker Hub** es el registry publico por defecto: gratuito para imagenes publicas, con planes pagos para privadas, y el destino donde vive la mayoria de imagenes oficiales de la comunidad. Para codigo propietario de una empresa, sin embargo, es mas comun usar un **registry privado** integrado con la infraestructura cloud (ECR si se usa AWS, GCR/Artifact Registry si se usa GCP) por la integracion nativa con permisos IAM, o **Harbor** como opcion self-hosted con features de seguridad empresarial (escaneo integrado, control de acceso granular, replicacion).

La eleccion no es solo tecnica: involucra costos (transferencia de datos, almacenamiento), latencia (un registry en la misma region/nube que los servidores de deploy es mas rapido), y control (un registry privado da mas control sobre politicas de retencion y acceso que dependen de terceros).

## Modelo mental

Pensa Docker Hub como una plaza publica de mercado, ideal para productos que querés que cualquiera pueda encontrar y usar. Un registry privado es como el deposito interno de tu propia empresa: solo tu equipo tiene la llave, y esta ubicado estrategicamente cerca de donde realmente se necesita el producto (tus servidores de produccion).

## Ejemplo base

```bash
# Login y push a Docker Hub
docker login
docker tag mi-imagen usuario/mi-imagen:1.0
docker push usuario/mi-imagen:1.0

# Login y push a un registry privado (ejemplo generico)
docker login mi-registro-privado.com
docker tag mi-imagen mi-registro-privado.com/proyecto/mi-imagen:1.0
docker push mi-registro-privado.com/proyecto/mi-imagen:1.0
```

## Como se usa en un proyecto real

1. Se usan imagenes publicas de Docker Hub como bases (Node, Postgres, Nginx oficiales) libremente, dado que son de acceso publico y ampliamente auditadas.
2. Se aloja el codigo propietario de la empresa en un registry privado, alineado con el proveedor cloud usado para minimizar latencia y costos de transferencia entre registry y servidores de deploy.
3. Se evalua Harbor u otra opcion self-hosted cuando hay requisitos de compliance que exigen control total sobre donde vive el codigo.
4. Se configuran credenciales de acceso al registry privado como secretos gestionados (nunca hardcodeados) en el pipeline de CI/CD.
5. Se documenta que registry aloja cada tipo de imagen del proyecto (bases publicas vs codigo propio) para claridad del equipo.

## Criterio junior

Un junior debe poder hacer login, tag y push de una imagen tanto a Docker Hub como a un registry privado basico siguiendo instrucciones.

## Criterio mid

Un mid debe elegir el registry apropiado segun el contexto del proyecto (proveedor cloud usado, requisitos de privacidad) y configurar el acceso de forma segura en CI/CD.

## Criterio senior

Un senior debe definir la estrategia de registries de la organizacion completa, considerando costos de transferencia, latencia, compliance, y la integracion con el resto de la infraestructura cloud.

## Errores comunes

- Alojar codigo propietario sensible en un registry publico sin marcarlo explicitamente como privado.
- No considerar la latencia y costo de transferencia entre el registry elegido y los servidores donde se despliega, especialmente entre proveedores distintos.
- Hardcodear credenciales de acceso al registry privado en scripts o pipelines en vez de usar secretos gestionados.
- No documentar claramente que registry corresponde a cada tipo de imagen del proyecto.

## Practica

- [ ] Publica una imagen de practica en Docker Hub como repositorio privado.
- [ ] Configura acceso a un registry privado (ECR, GCR o equivalente) y publica una imagen ahi.
- [ ] Compara el costo y la latencia estimada de ambas opciones para tu contexto real.
- [ ] Configura las credenciales de acceso al registry como secreto en un pipeline de CI de ejemplo.
- [ ] Documenta que registry usarias para cada tipo de imagen en un proyecto real.

## Preguntas de repaso

- Que diferencia de proposito hay entre Docker Hub y un registry privado como ECR o GCR?
- Que factores de costo y latencia deberian influir en la eleccion de un registry?
- Por que las credenciales de acceso a un registry privado nunca deberian hardcodearse en un pipeline?
- Cuando se justificaria evaluar una opcion self-hosted como Harbor sobre un registry gestionado por el proveedor cloud?
