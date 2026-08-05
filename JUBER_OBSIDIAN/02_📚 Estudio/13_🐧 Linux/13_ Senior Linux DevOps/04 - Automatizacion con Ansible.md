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

# 04 - Automatizacion con Ansible

## Que vas a aprender

En este capitulo vas a aprender **Ansible**, la herramienta de automatizacion de configuracion mas usada en infraestructura Linux, completando el par visto en el tema de IaC: Terraform declara que infraestructura existe, Ansible declara que configuracion tiene cada servidor.

## Explicacion

Ansible ejecuta tareas sobre servidores remotos via **SSH** (sin necesitar instalar ningun agente en los servidores gestionados, a diferencia de otras herramientas de configuracion), definidas en archivos **YAML** llamados **playbooks**. Cada tarea es **idempotente** por diseño: ejecutar el mismo playbook multiples veces produce el mismo resultado final sin efectos secundarios acumulativos, a diferencia de un script Bash tradicional que podria fallar o comportarse distinto en una segunda ejecucion.

Este es exactamente el mismo principio de scripts robustos ya visto en el capitulo de Scripting con Bash de este libro, pero llevado a escala: en vez de ejecutar comandos manualmente o via SSH uno por uno en cada servidor, un playbook de Ansible describe el estado deseado ("nginx debe estar instalado y corriendo", "este archivo de configuracion debe tener este contenido"), y Ansible se encarga de verificar el estado actual y aplicar solo los cambios necesarios para llegar al estado declarado.

## Modelo mental

Pensa un playbook de Ansible como una lista de verificacion de un inspector de calidad: en vez de decir "hace estos pasos en este orden exacto" (como un script tradicional), dice "este es el resultado final que debe existir", y el inspector revisa cada servidor, corrige solo lo que falta, sin repetir trabajo innecesario en lo que ya esta correcto.

## Ejemplo base

```yaml
# playbook.yml
- hosts: servidores_web
  become: true
  tasks:
    - name: Instalar nginx
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: Copiar configuracion
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: reiniciar nginx

    - name: Asegurar que nginx esta corriendo
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: reiniciar nginx
      service:
        name: nginx
        state: restarted
```

```bash
ansible-playbook -i inventario.ini playbook.yml    # ejecuta el playbook sobre los servidores del inventario
ansible-playbook playbook.yml --check                # dry-run: muestra que cambiaria sin aplicarlo
```

## Como se usa en un proyecto real

1. Se usa Ansible para configurar servidores nuevos de forma consistente, aplicando el mismo playbook a cualquier cantidad de servidores simultaneamente.
2. Se aprovecha `--check` (dry-run) para verificar que cambios aplicaria un playbook antes de ejecutarlo realmente, el mismo principio de "probar antes de aplicar" ya visto en varios capitulos anteriores.
3. Se organizan playbooks por rol (servidor web, base de datos, aplicacion) reutilizables entre distintos proyectos, en vez de scripts monoliticos dificiles de mantener.
4. Se versiona todo el codigo de Ansible en el mismo repositorio (o uno dedicado) que la infraestructura de Terraform, completando el ciclo de infraestructura como codigo del tema anterior.
5. Se usa Ansible para tareas de mantenimiento recurrentes en flotas de servidores (aplicar actualizaciones de seguridad, ver capitulo correspondiente, en todos los servidores a la vez de forma consistente).

## Criterio junior

Un junior debe poder ejecutar un playbook de Ansible ya escrito, y entender el concepto de idempotencia.

## Criterio mid

Un mid debe escribir playbooks propios para configurar servicios basicos, usando `--check` para verificar cambios antes de aplicarlos.

## Criterio senior

Un senior debe diseñar la estrategia completa de automatizacion de configuracion de la organizacion con Ansible, organizando playbooks reutilizables por rol, y coordinando con Terraform para un flujo completo de infraestructura como codigo.

## Errores comunes

- Escribir tareas de Ansible que no son verdaderamente idempotentes (usando comandos shell crudos en vez de modulos nativos de Ansible), perdiendo la garantia de comportamiento consistente entre ejecuciones.
- No usar `--check` antes de ejecutar un playbook en produccion, aplicando cambios sin haber verificado su impacto real primero.
- Escribir playbooks monoliticos dificiles de mantener en vez de organizarlos por rol reutilizable.
- No versionar el codigo de Ansible junto con el resto de la infraestructura como codigo, perdiendo historial y capacidad de revision colaborativa.

## Practica

- [ ] Escribe un playbook simple que instale y configure un servicio (como nginx) en un servidor de practica.
- [ ] Ejecuta ese playbook con `--check` antes de aplicarlo realmente.
- [ ] Ejecuta el mismo playbook dos veces seguidas y verifica que la segunda ejecucion no genera cambios innecesarios (idempotencia).
- [ ] Organiza tareas relacionadas en un rol reutilizable de Ansible.
- [ ] Documenta como conectarias Terraform (infraestructura) con Ansible (configuracion) en un flujo completo.

## Preguntas de repaso

- Que significa que una tarea de Ansible sea idempotente, y por que eso es valioso?
- Que ventaja da Ansible sobre SSH manual o scripts Bash tradicionales para gestionar multiples servidores?
- Para que sirve la opcion `--check` de `ansible-playbook`?
- Como se complementan Terraform y Ansible en un flujo completo de infraestructura como codigo?
