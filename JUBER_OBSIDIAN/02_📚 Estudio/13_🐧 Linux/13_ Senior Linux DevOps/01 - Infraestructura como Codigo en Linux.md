---
tags:
  - Linux
  - DevOps
  - Linux-Senior
Creado: 2026-07-28
Relacionado:
  - Senior Linux DevOps
  - LINUX
---

# 01 - Infraestructura como Codigo en Linux

## Que vas a aprender

En este capitulo vas a consolidar el principio de **infraestructura como codigo (IaC)** aplicado a servidores Linux: por que la configuracion manual de servidores es un riesgo a largo plazo, y como herramientas de aprovisionamiento (Terraform para infraestructura, y Ansible en el siguiente tema para configuracion) resuelven ese problema.

## Explicacion

Un servidor configurado manualmente, comando por comando a lo largo del tiempo, se convierte en lo que se conoce como **servidor mascota**: unico, irreproducible, y donde nadie recuerda exactamente todos los pasos que lo llevaron a su estado actual. Esto conecta directamente con el principio de inmutabilidad ya visto en el capitulo Senior del libro de Docker: asi como un container nunca deberia modificarse en caliente, un servidor tampoco deberia depender de configuracion manual acumulada sin registro.

**Terraform** (infraestructura declarativa: que servidores, redes y recursos cloud deben existir) y **Ansible** (configuracion declarativa: que software y configuracion debe tener cada servidor) son las herramientas mas comunes para aplicar IaC. La idea central es que toda la infraestructura este descrita en archivos versionados en un repositorio, de forma que reconstruir un servidor desde cero (ante una falla, o para crear un ambiente identico de staging) sea un proceso automatizado y reproducible, no un ejercicio de memoria de un administrador especifico.

## Modelo mental

Pensa un servidor configurado manualmente como una receta de cocina que existe solo en la memoria de un unico chef, modificada improvisadamente con el tiempo: si ese chef se va, nadie puede reproducir exactamente el mismo plato. Infraestructura como codigo es escribir esa receta completa y exacta en un papel (codigo versionado) que cualquier otro chef puede seguir para producir exactamente el mismo resultado, sin importar quien la ejecute.

## Ejemplo base

```hcl
# Terraform: declara que infraestructura debe existir (ejemplo conceptual simplificado)
resource "aws_instance" "servidor_api" {
  ami           = "ami-12345"
  instance_type = "t3.medium"
  tags = { Name = "servidor-api-produccion" }
}
```

```bash
terraform plan     # muestra que cambios se aplicarian, sin aplicarlos todavia
terraform apply     # aplica los cambios declarados
```

## Como se usa en un proyecto real

1. Se declara toda infraestructura nueva (servidores, redes, almacenamiento) en codigo versionado, evitando creacion manual de recursos que despues nadie puede reproducir.
2. Se usa `terraform plan` siempre antes de `apply`, revisando exactamente que cambios se van a realizar antes de confirmarlos, similar al principio de "probar antes de aplicar" visto en el capitulo de Bash Scripting.
3. Se versiona el codigo de infraestructura en el mismo tipo de flujo de control de versiones (revision, PR) que el codigo de aplicacion.
4. Se conecta este principio directamente con la inmutabilidad de containers vista en el libro de Docker: ambos resuelven el mismo problema fundamental (reproducibilidad, auditabilidad) en capas distintas del stack.
5. Se documenta con ADRs (ver capitulo Senior del libro de NestJS) las decisiones de arquitectura de infraestructura significativas tomadas al diseñar el codigo de IaC.

## Criterio junior

Un junior debe entender el concepto de "servidor mascota" versus infraestructura reproducible, y poder leer codigo Terraform basico.

## Criterio mid

Un mid debe escribir configuracion Terraform simple para recursos basicos, siguiendo el flujo `plan` antes de `apply` de forma disciplinada.

## Criterio senior

Un senior debe diseñar la estrategia completa de IaC de la organizacion, versionando toda la infraestructura, y liderar la migracion de servidores mascota existentes hacia infraestructura reproducible.

## Errores comunes

- Crear recursos de infraestructura manualmente "por esta vez" y despues perder registro de esa creacion, generando drift entre lo declarado en codigo y lo que realmente existe.
- No revisar `terraform plan` antes de `apply`, aplicando cambios sin haber verificado exactamente que se va a modificar.
- No versionar el codigo de infraestructura con el mismo rigor que el codigo de aplicacion, perdiendo historial y capacidad de revision.
- Mantener servidores mascota criticos sin ningun plan de migracion hacia infraestructura reproducible.

## Practica

- [ ] Investiga la sintaxis basica de Terraform (o una herramienta equivalente) para declarar un recurso simple.
- [ ] Practica el flujo `plan` antes de `apply` con un cambio de infraestructura de ejemplo.
- [ ] Identifica un "servidor mascota" real o hipotetico en tu experiencia y diseña como lo convertirias en infraestructura reproducible.
- [ ] Conecta este principio con la inmutabilidad de containers ya vista en el libro de Docker.
- [ ] Documenta una decision de infraestructura significativa como un ADR.

## Preguntas de repaso

- Que es un "servidor mascota" y por que representa un riesgo a largo plazo?
- Que diferencia hay entre lo que declara Terraform (infraestructura) y lo que configura Ansible (software dentro de esa infraestructura)?
- Por que es importante revisar `terraform plan` antes de aplicar cambios?
- Como se conecta el principio de IaC con la inmutabilidad de containers vista en el libro de Docker?
