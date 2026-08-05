---
tags:
  - Linux
  - Paquetes
  - Linux-Paquetes
Creado: 2026-07-28
Relacionado:
  - Gestion de Paquetes y Software
  - LINUX
---

# 02 - YUM DNF y RedHat CentOS

## Que vas a aprender

En este capitulo vas a aprender la otra gran familia de gestion de paquetes de Linux: **YUM** (tradicional) y **DNF** (su sucesor moderno), usados en RedHat, CentOS, Fedora, Rocky Linux y AlmaLinux, con equivalencias directas a lo ya visto con APT.

## Explicacion

DNF (Dandified YUM) reemplazo a YUM como gestor de paquetes por defecto en distribuciones modernas de esta familia, resolviendo dependencias de forma mas eficiente y con mejor manejo de conflictos, aunque la sintaxis de comandos es practicamente identica, facilitando la transicion. Ambos gestionan paquetes `.rpm` (el equivalente a `.deb` en Debian/Ubuntu) y usan repositorios definidos en `/etc/yum.repos.d/`.

La correspondencia con APT es casi uno a uno: `dnf install` equivale a `apt install`, `dnf remove` a `apt remove`, `dnf search` a `apt search`. Un concepto adicional relevante en esta familia es el uso comun de **repositorios EPEL** (Extra Packages for Enterprise Linux), que agrega paquetes adicionales no incluidos en los repositorios base de RHEL/CentOS por defecto, similar en espiritu a agregar un PPA en Ubuntu.

## Modelo mental

Pensa DNF como el mismo tipo de asistente de compras que APT, pero trabajando en una tienda distinta con productos empaquetados de forma diferente (`.rpm` en vez de `.deb`): el servicio que presta es conceptualmente identico, solo cambia el formato del catalogo y algunos detalles de implementacion interna.

## Ejemplo base

```bash
sudo dnf check-update              # equivalente a "apt update", solo revisa, no instala
sudo dnf install nginx              # instala nginx y sus dependencias
sudo dnf remove nginx                # desinstala

dnf search "servidor web"            # busca paquetes por palabra clave
dnf info nginx                       # informacion detallada, equivalente a "apt show"

sudo dnf autoremove                  # limpia dependencias huerfanas

# Habilitar EPEL para acceder a paquetes adicionales
sudo dnf install epel-release
```

## Como se usa en un proyecto real

1. Se usa DNF en cualquier servidor RHEL, CentOS, Fedora, Rocky Linux o AlmaLinux, aplicando el mismo criterio de "actualizar catalogo antes de instalar" ya visto con APT.
2. Se habilita el repositorio EPEL cuando se necesita software adicional no incluido en los repositorios base, comun en entornos empresariales que usan RHEL/derivados.
3. Se documenta claramente que familia de distribucion (Debian/Ubuntu vs RedHat/derivados) asume cada script de instalacion, evitando confusion al aplicar comandos de la familia equivocada.
4. Se traduce mentalmente la equivalencia entre APT y DNF al trabajar en infraestructura mixta que usa ambas familias de distribuciones.
5. Se verifica la confiabilidad de cualquier repositorio adicional habilitado, con el mismo criterio de seguridad aplicado a APT y a imagenes Docker.

## Criterio junior

Un junior debe poder instalar y desinstalar paquetes basicos con DNF, reconociendo su equivalencia directa con los comandos de APT.

## Criterio mid

Un mid debe habilitar repositorios adicionales (como EPEL) de forma segura, y traducir comandos entre APT y DNF al trabajar en infraestructura mixta.

## Criterio senior

Un senior debe estandarizar documentacion de instalacion que cubra ambas familias de distribuciones cuando la infraestructura del equipo es mixta, evitando scripts que asuman incorrectamente una familia especifica.

## Errores comunes

- Intentar usar comandos de APT en un servidor RedHat/CentOS (o viceversa) sin adaptar la sintaxis, un error comun al copiar guias sin verificar la distribucion real.
- No habilitar EPEL cuando se necesita software adicional, sin saber que ciertos paquetes comunes no vienen en los repositorios base de RHEL/derivados.
- No documentar claramente que familia de distribucion asume un script o guia, generando confusion cuando se aplica al sistema equivocado.
- No verificar la confiabilidad de repositorios adicionales habilitados con `dnf`, con el mismo riesgo que agregar repositorios no confiables en APT.

## Practica

- [ ] Si tenes acceso a un sistema RHEL/CentOS/Fedora, instala un paquete simple con DNF.
- [ ] Compara la sintaxis de DNF con la de APT ya aprendida, identificando la equivalencia directa de cada comando.
- [ ] Habilita el repositorio EPEL y busca un paquete que solo este disponible ahi.
- [ ] Documenta una guia de instalacion que cubra tanto APT como DNF para un mismo software.
- [ ] Anota que consideracion de seguridad aplicarias al habilitar un repositorio adicional en DNF.

## Preguntas de repaso

- Que diferencia hay entre YUM y DNF, y por que DNF lo reemplazo como default?
- Que equivalente de APT es `dnf install`, y cual es `dnf search`?
- Que es el repositorio EPEL y para que se usa?
- Que riesgo de seguridad comparte habilitar un repositorio adicional en DNF con agregar un repositorio de terceros en APT?
