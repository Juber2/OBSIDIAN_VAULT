---
tags:
  - Linux
  - Sistemas
  - DevOps
  - Libro
Creado: 2026-07-28
Relacionado:
  - ESTUDIO
---

#Principal

# Libro Linux Junior a Senior

Esta carpeta no es solo una ruta. Esta pensada como un **libro de estudio practico** para avanzar desde bases de administracion de sistemas hasta criterio senior de infraestructura Linux.

La estructura esta dividida en capitulos. Cada capitulo contiene explicacion, modelo mental, ejemplos, errores comunes, practica, preguntas de repaso y criterios junior/mid/senior.

## Capitulos

| Orden | Capitulo | Estado | Nota |
|---|---|---|---|
| 1 | Fundamentos del Sistema | Pendiente | [[Fundamentos del Sistema]] |
| 2 | Linea de Comandos y Shell | Pendiente | [[Linea de Comandos y Shell]] |
| 3 | Gestion de Archivos y Permisos | Pendiente | [[Gestion de Archivos y Permisos]] |
| 4 | Procesos y Gestion del Sistema | Pendiente | [[Procesos y Gestion del Sistema]] |
| 5 | Editores y Manipulacion de Texto | Pendiente | [[Editores y Manipulacion de Texto]] |
| 6 | Scripting con Bash | Pendiente | [[Scripting con Bash]] |
| 7 | Redes | Pendiente | [[Redes]] |
| 8 | Gestion de Paquetes y Software | Pendiente | [[Gestion de Paquetes y Software]] |
| 9 | Almacenamiento y Sistemas de Archivos | Pendiente | [[Almacenamiento y Sistemas de Archivos]] |
| 10 | Seguridad | Pendiente | [[Seguridad]] |
| 11 | Administracion de Servidores | Pendiente | [[Administracion de Servidores]] |
| 12 | Performance y Troubleshooting | Pendiente | [[Performance y Troubleshooting]] |
| 13 | Senior Linux DevOps | Pendiente | [[Senior Linux DevOps]] |
| 14 | Proyectos Linux | Pendiente | [[Proyectos]] |

## Como usar este libro

1. Lee el capitulo en orden.
2. Practica cada comando en un entorno real: WSL2, una VM, o un servidor cloud economico (ver capitulo de Fundamentos).
3. Convierte cada tema importante en una modificacion real de un servidor de practica.
4. Aplica lo aprendido a la infraestructura de tus proyectos reales, conectando con [[NESTJS|el backend NestJS]] y [[DOCKER|Docker]].
5. Vuelve a cada nota y marca el checklist solo cuando puedas explicarlo sin mirar tutorial.

## Resultado esperado

Al terminar, deberias poder administrar servidores Linux reales de punta a punta: terminal y scripting solido, permisos y usuarios correctamente gestionados, procesos y servicios administrados con systemd, redes y firewall configurados con criterio, almacenamiento diseñado para crecimiento, seguridad endurecida (SSH, sudo, MAC, auditoria), servidores web y bases de datos administrados, troubleshooting de performance con herramientas profundas, e infraestructura reproducible con Ansible/Terraform.

## Checkpoint senior

- [ ] Domino la terminal y Bash scripting lo suficiente para automatizar cualquier tarea administrativa repetitiva.
- [ ] Puedo diseñar permisos, usuarios y sudo aplicando el principio de menor privilegio en cualquier escenario.
- [ ] Puedo administrar servicios de larga vida con systemd, con reinicio automatico y dependencias correctas.
- [ ] Puedo diagnosticar problemas de performance (CPU, memoria, disco, red) con evidencia real, no suposiciones.
- [ ] Puedo endurecer la seguridad de un servidor (SSH, sudo, MAC, actualizaciones) con criterio de defensa en profundidad.
- [ ] Puedo diseñar infraestructura reproducible con Ansible/Terraform, sin depender de servidores mascota.
- [ ] Puedo explicar como los mismos mecanismos del kernel (namespaces, cgroups) sustentan la containerizacion vista en el libro de Docker.

## Referencias base

- [The Linux Documentation Project](https://tldp.org/)
- [Arch Wiki](https://wiki.archlinux.org/) (referencia tecnica excelente aunque uses otra distro)
- [systemd Documentation](https://www.freedesktop.org/wiki/Software/systemd/)
- [Ansible Documentation](https://docs.ansible.com/)
