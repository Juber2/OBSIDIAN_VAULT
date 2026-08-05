---
tags:
  - Linux
  - Sistemas
  - Linux-Fundamentos
Creado: 2026-07-28
Relacionado:
  - Fundamentos del Sistema
  - LINUX
---

# 05 - Instalacion y Primeros Pasos

## Que vas a aprender

En este capitulo vas a aprender las distintas formas practicas de tener un entorno Linux para practicar y trabajar: **WSL2** en Windows, **maquinas virtuales**, y **servidores cloud**, junto con la configuracion minima inicial que cualquier sistema nuevo necesita.

## Explicacion

Para un desarrollador que usa Windows, **WSL2** (Windows Subsystem for Linux) da un kernel Linux real corriendo integrado con Windows, siendo la opcion mas practica para aprender y desarrollar sin necesitar una maquina separada (y es ademas la base sobre la que corre Docker Desktop, ya visto en el libro de Docker). Una **maquina virtual** (VirtualBox, VMware, Hyper-V) da un entorno completamente aislado, util para practicar administracion de sistema de forma mas realista, incluyendo el proceso de instalacion completo. Un **servidor cloud** (una instancia EC2, un droplet de DigitalOcean) es la experiencia mas cercana a produccion real, con acceso remoto por SSH desde el primer momento.

Independientemente de la via elegida, los primeros pasos en cualquier sistema Linux nuevo siguen un patron: actualizar el sistema, crear un usuario no-root con permisos de sudo, configurar el firewall basico, y asegurar el acceso SSH (ver capitulo de Redes y Seguridad).

## Modelo mental

Pensa estas opciones como distintas formas de conseguir tiempo de manejo antes de salir a la ruta real: WSL2 es un simulador integrado comodo para practicar todos los dias, una VM es un auto de practica en un circuito cerrado similar a condiciones reales, y un servidor cloud es directamente salir a la ruta con trafico real, la experiencia mas parecida a lo que enfrentaras en un trabajo real.

## Ejemplo base

```bash
# Primeros pasos estandar en un servidor Linux nuevo
sudo apt update && sudo apt upgrade -y     # actualizar el sistema (Debian/Ubuntu)

sudo adduser nuevo_usuario                 # crear usuario no-root
sudo usermod -aG sudo nuevo_usuario        # darle permisos de sudo

sudo ufw allow OpenSSH                     # permitir SSH en el firewall
sudo ufw enable                            # activar el firewall
```

## Como se usa en un proyecto real

1. Se usa WSL2 para el flujo de trabajo diario de desarrollo, dado que se integra naturalmente con herramientas ya conocidas (Docker, editores) sin necesitar cambiar de maquina.
2. Se usa una VM cuando se necesita practicar procesos que modifican el sistema de forma mas profunda (particionamiento, instalacion desde cero) sin riesgo sobre el sistema principal.
3. Se usa un servidor cloud economico como entorno de practica realista para administracion remota, replicando el escenario real de un trabajo en produccion.
4. Se crea siempre un usuario no-root con sudo como primer paso en cualquier servidor nuevo, evitando trabajar directamente como root en el dia a dia.
5. Se configura el firewall basico y se asegura el acceso SSH desde el primer momento, nunca dejando un servidor recien creado expuesto sin configuracion minima de seguridad.

## Criterio junior

Un junior debe poder configurar WSL2 o una VM basica, y realizar los primeros pasos estandar (actualizar sistema, crear usuario con sudo) en un servidor nuevo.

## Criterio mid

Un mid debe configurar un servidor cloud desde cero, incluyendo firewall basico y hardening inicial de SSH, siguiendo un checklist estandar de primeros pasos.

## Criterio senior

Un senior debe automatizar el proceso de configuracion inicial de servidores nuevos (via scripts o herramientas de infraestructura como codigo, ver capitulo Senior) para que sea reproducible y consistente en toda la infraestructura del equipo.

## Errores comunes

- Trabajar directamente como root en el dia a dia en vez de crear un usuario no-root con sudo desde el principio.
- Dejar un servidor recien creado sin configuracion basica de firewall, expuesto innecesariamente.
- No actualizar el sistema en los primeros pasos, arrancando con paquetes desactualizados que pueden tener vulnerabilidades conocidas.
- Repetir manualmente los mismos pasos de configuracion inicial en cada servidor nuevo sin automatizarlos.

## Practica

- [ ] Configura WSL2 en Windows o una VM Linux basica si aun no tenes un entorno de practica.
- [ ] Actualiza el sistema y crea un usuario no-root con permisos de sudo.
- [ ] Configura un firewall basico permitiendo solo SSH inicialmente.
- [ ] Si tenes acceso a un servidor cloud, repite este proceso completo ahi.
- [ ] Documenta tu propio checklist de "primeros pasos" para cualquier servidor Linux nuevo.

## Preguntas de repaso

- Que ventaja practica tiene WSL2 para un desarrollador que ya usa Windows a diario?
- Por que crear un usuario no-root con sudo deberia ser de los primeros pasos en cualquier servidor nuevo?
- Que configuracion minima de seguridad aplicarias antes de considerar "listo" un servidor recien creado?
- Como automatizarias el proceso de configuracion inicial para que sea reproducible entre servidores?
