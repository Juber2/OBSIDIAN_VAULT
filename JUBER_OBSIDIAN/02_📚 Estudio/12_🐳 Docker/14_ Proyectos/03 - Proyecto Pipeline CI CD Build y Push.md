---
tags:
  - Docker
  - DevOps
  - Docker-Proyectos
Creado: 2026-07-28
Relacionado:
  - Proyectos
  - DOCKER
---

# 03 - Proyecto Pipeline CI CD Build y Push

## Que vas a construir

Un pipeline de CI/CD (GitHub Actions u otra plataforma) que construya, testee y publique automaticamente la imagen del proyecto, aplicando los capitulos de CI/CD y Registries.

## Alcance funcional

- Pipeline disparado en pull requests (build + test, sin publicar) y en merges a la rama principal (build + test + publish).
- Estrategia de tagging con hash de commit como tag trazable primario.
- Cache remoto configurado para acelerar builds sucesivos en el pipeline.
- Escaneo de vulnerabilidades (Trivy o Docker Scout) integrado como paso del pipeline.
- Credenciales del registry configuradas como secretos, nunca en texto plano.

## Checklist tecnica

- [ ] Un pull request dispara build y test, pero nunca publica una imagen al registry.
- [ ] Un merge a la rama principal publica la imagen con el tag de commit correspondiente.
- [ ] El cache remoto reduce medibleente el tiempo de build en ejecuciones sucesivas con cambios menores.
- [ ] El escaneo de vulnerabilidades reporta el resultado como parte del pipeline, con un umbral de severidad definido.
- [ ] Las credenciales del registry nunca aparecen en logs del pipeline.

## Criterio de exito

El proyecto se considera completo cuando un cambio de codigo mergeado a la rama principal resulta automaticamente en una imagen nueva, correctamente tageada y trazable hasta el commit exacto, publicada en el registry sin ninguna intervencion manual, con evidencia de que paso por lint, tests y escaneo de seguridad.

## Errores comunes a evitar

- Publicar imagenes desde pull requests, ensuciando el registry con builds de trabajo en progreso.
- No configurar cache remoto, dejando que cada ejecucion del pipeline reconstruya todo desde cero innecesariamente.
- Ignorar o no revisar el resultado del escaneo de vulnerabilidades, dejandolo como un paso decorativo sin efecto real.

## Extension opcional

Agregar firma de la imagen con cosign como ultimo paso del pipeline, y documentar como se verificaria esa firma antes de un despliegue.
