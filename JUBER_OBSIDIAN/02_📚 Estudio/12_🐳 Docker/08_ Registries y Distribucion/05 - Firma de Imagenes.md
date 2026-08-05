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

# 05 - Firma de Imagenes

## Que vas a aprender

En este capitulo vas a aprender el concepto de **firma criptografica de imagenes** (Docker Content Trust y **cosign**), como mecanismo para verificar que una imagen desplegada es exactamente la que el pipeline de CI construyo, sin alteraciones intermedias.

## Explicacion

Sin firma de imagenes, cualquiera con acceso de push al registry (o un atacante que comprometa esas credenciales) podria reemplazar una imagen legitima por una maliciosa bajo el mismo tag, y nada en el sistema lo detectaria automaticamente. La **firma criptografica** resuelve esto: el pipeline de CI firma la imagen con una clave privada al momento de publicarla, y los servidores de deploy (o el orquestador) pueden verificar esa firma con la clave publica correspondiente antes de correr la imagen, rechazando cualquier imagen no firmada o firmada incorrectamente.

**Docker Content Trust** (basado en Notary) fue el mecanismo original de Docker; **cosign** (parte del proyecto Sigstore) es la herramienta moderna mas adoptada actualmente, con soporte para firma sin necesidad de gestionar claves manualmente (keyless signing, usando identidad OIDC) ademas del modo tradicional con claves.

## Modelo mental

Pensa la firma de una imagen como un sello de cera en un sobre oficial: cualquiera podria intentar fabricar un sobre parecido, pero solo quien tiene el sello original (la clave privada) puede producir la marca exacta que el receptor (el servidor de deploy) reconoce como autentica antes de aceptar el contenido.

## Ejemplo base

```bash
# Firmar una imagen con cosign despues de publicarla
cosign sign --key cosign.key mi-registro/mi-imagen:1.0

# Verificar la firma antes de desplegar
cosign verify --key cosign.pub mi-registro/mi-imagen:1.0

# Keyless signing usando identidad OIDC del pipeline de CI (sin gestionar claves manualmente)
cosign sign mi-registro/mi-imagen:1.0
```

## Como se usa en un proyecto real

1. Se firma cada imagen automaticamente como ultimo paso del pipeline de CI, inmediatamente despues de publicarla al registry.
2. Se configura el orquestador o el proceso de deploy para verificar la firma antes de correr cualquier imagen, rechazando las no firmadas o firmadas incorrectamente.
3. Se evalua keyless signing (identidad OIDC) para evitar la complejidad operativa de gestionar y rotar claves privadas de firma manualmente.
4. Se documenta el proceso de verificacion como parte del pipeline de deploy, visible para todo el equipo de infraestructura.
5. Se combina la firma con el escaneo de vulnerabilidades (ver capitulo de Seguridad) como capas complementarias de confianza en la cadena de suministro.

## Criterio junior

Un junior debe entender conceptualmente por que firmar una imagen previene que una version alterada o maliciosa sea desplegada sin ser detectada.

## Criterio mid

Un mid debe integrar la firma de imagenes como paso automatico del pipeline de CI, y configurar la verificacion antes del deploy.

## Criterio senior

Un senior debe evaluar y adoptar firma de imagenes como parte de la estrategia de seguridad de cadena de suministro de toda la organizacion, decidiendo entre firma con claves gestionadas o keyless signing segun la madurez operativa del equipo.

## Errores comunes

- No firmar imagenes en absoluto, dejando el sistema vulnerable a que una imagen alterada o maliciosa se despliegue sin deteccion.
- Firmar imagenes pero no verificar la firma antes de desplegar, haciendo que la firma exista pero no aporte proteccion real.
- Gestionar claves de firma manualmente sin un proceso claro de rotacion, arriesgando que una clave comprometida quede activa indefinidamente.
- No integrar la firma como paso automatico del pipeline, dependiendo de que alguien se acuerde de firmar manualmente cada release.

## Practica

- [ ] Instala cosign y firma una imagen de practica publicada en un registry.
- [ ] Verifica esa firma antes de "desplegar" (simular el paso) la imagen.
- [ ] Integra la firma como paso automatico de un pipeline de CI de ejemplo.
- [ ] Investiga el flujo de keyless signing con identidad OIDC y compara su complejidad operativa con claves tradicionales.
- [ ] Documenta como tu proceso de deploy verificaria (o deberia verificar) la firma antes de correr una imagen.

## Preguntas de repaso

- Que problema de seguridad resuelve firmar imagenes que el escaneo de vulnerabilidades por si solo no resuelve?
- Que diferencia hay entre firmar una imagen y verificarla, y por que ambos pasos son necesarios?
- Que ventaja operativa da el keyless signing sobre la gestion tradicional de claves privadas?
- En que punto del pipeline de CI/CD integrarias la firma y la verificacion de imagenes?
