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

# 03 - Push Pull y Autenticacion

## Que vas a aprender

En este capitulo vas a profundizar en el flujo de **autenticacion contra registries**: tokens de acceso vs usuario/contraseña, autenticacion en CI/CD sin exponer credenciales, y buenas practicas al dar permisos de push/pull.

## Explicacion

Autenticarse contra un registry (`docker login`) tradicionalmente usaba usuario y contraseña, pero la practica moderna recomendada es usar **tokens de acceso de vida limitada y con permisos acotados** (por ejemplo, un token que solo puede hacer `pull`, sin `push`, para servidores que solo consumen imagenes), en vez de credenciales de cuenta completas con todos los permisos.

En pipelines de CI/CD, las credenciales del registry nunca deben aparecer en texto plano en el codigo del pipeline; se configuran como **secretos** del sistema de CI (variables de entorno cifradas, integraciones con vault), y muchos proveedores cloud ofrecen autenticacion basada en **identidad del propio pipeline** (OIDC, roles IAM) que elimina la necesidad de credenciales estaticas por completo.

## Modelo mental

Pensa un token de acceso limitado como una llave que solo abre una puerta especifica (por ejemplo, solo la puerta de "sacar cosas", nunca la de "meter cosas nuevas"), en vez de darle a cada servidor la llave maestra completa de tu cuenta, que si se filtra, compromete todo.

## Ejemplo base

```bash
# Token de acceso con permisos acotados en vez de contraseña de cuenta completa
docker login mi-registro.com -u usuario --password-stdin <<< "$REGISTRY_TOKEN"

# En CI, el secreto nunca aparece en texto plano en el script
echo "$REGISTRY_TOKEN" | docker login mi-registro.com -u usuario --password-stdin
```

```yaml
# Ejemplo conceptual: autenticacion via identidad del pipeline (OIDC), sin secreto estatico
permissions:
  id-token: write
steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789:role/ci-push-role
```

## Como se usa en un proyecto real

1. Se usan tokens de acceso con permisos minimos necesarios (solo pull para servidores de produccion que consumen imagenes, push solo para el pipeline de CI que las publica) en vez de credenciales de cuenta completas.
2. Se configuran las credenciales del registry como secretos gestionados por el sistema de CI, nunca visibles en logs ni en el codigo del pipeline.
3. Se evalua autenticacion basada en identidad (OIDC con roles IAM) cuando el proveedor cloud lo soporta, eliminando la necesidad de rotar credenciales estaticas manualmente.
4. Se rota periodicamente cualquier token de acceso de larga duracion que no pueda reemplazarse por autenticacion basada en identidad.
5. Se audita quien tiene permisos de push sobre cada repositorio del registry, limitando ese acceso a los pipelines automatizados y personas estrictamente necesarias.

## Criterio junior

Un junior debe poder autenticarse contra un registry usando un token en vez de contraseña de cuenta, siguiendo el flujo de `docker login`.

## Criterio mid

Un mid debe configurar credenciales de registry como secretos en un pipeline de CI, asegurando que nunca aparezcan en texto plano en logs o codigo.

## Criterio senior

Un senior debe migrar la autenticacion de credenciales estaticas a autenticacion basada en identidad (OIDC/IAM roles) cuando sea posible, y definir la politica de permisos minimos de push/pull para todo el equipo y sus pipelines.

## Errores comunes

- Usar la contraseña de cuenta completa en vez de un token de acceso con permisos acotados para automatizaciones.
- Exponer credenciales del registry en logs de CI por no usar el mecanismo de secretos correctamente (por ejemplo, con `echo` visible en vez de `--password-stdin`).
- Dar permisos de push a servidores que solo necesitan hacer pull de imagenes ya publicadas.
- No rotar tokens de acceso de larga duracion periodicamente, aumentando el riesgo si alguno se filtra sin que nadie lo note a tiempo.

## Practica

- [ ] Genera un token de acceso con permisos acotados para un registry de practica.
- [ ] Configura ese token como secreto en un pipeline de CI, verificando que nunca aparece en los logs.
- [ ] Distingue en tu infraestructura que servidores necesitan solo permiso de pull vs cuales necesitan push.
- [ ] Investiga si tu proveedor cloud soporta autenticacion basada en identidad (OIDC) para eliminar credenciales estaticas.
- [ ] Audita quien tiene actualmente permisos de push sobre los repositorios de tu registry.

## Preguntas de repaso

- Por que un token de acceso con permisos acotados es mas seguro que usar la contraseña de cuenta completa?
- Como asegurarias que las credenciales del registry nunca aparezcan en texto plano en logs de CI?
- Que ventaja da la autenticacion basada en identidad (OIDC) sobre credenciales estaticas de larga duracion?
- Que servidores de tu infraestructura deberian tener solo permiso de pull, sin push?
