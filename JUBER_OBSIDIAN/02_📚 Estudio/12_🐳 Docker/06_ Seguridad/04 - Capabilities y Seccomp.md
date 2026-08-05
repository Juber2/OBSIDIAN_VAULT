---
tags:
  - Docker
  - DevOps
  - Docker-Seguridad
Creado: 2026-07-28
Relacionado:
  - Seguridad
  - DOCKER
---

# 04 - Capabilities y Seccomp

## Que vas a aprender

En este capitulo vas a profundizar en dos mecanismos de seguridad de bajo nivel que Docker usa para limitar lo que un container puede hacer en el kernel: **Linux capabilities** y **seccomp**, y como restringirlos mas alla de lo que Docker aplica por defecto.

## Explicacion

Tradicionalmente, root en Linux tiene **todos los privilegios** del sistema. Las **capabilities** dividen esos privilegios en piezas granulares (por ejemplo, `NET_BIND_SERVICE` para bindear puertos privilegiados, `SYS_ADMIN` para operaciones administrativas amplias), permitiendo otorgar solo las que un proceso realmente necesita. Docker ya restringe un conjunto razonable de capabilities por defecto (no da el set completo de root del host), pero se puede reducir aun mas explicitamente con `--cap-drop` (quitar) y `--cap-add` (agregar solo lo estrictamente necesario).

**Seccomp** (secure computing mode) filtra que **llamadas al sistema** (syscalls) puede hacer un proceso, independientemente de sus capabilities. Docker aplica un perfil seccomp por defecto que bloquea decenas de syscalls peligrosas o innecesarias para la mayoria de aplicaciones, reduciendo la superficie de ataque disponible incluso si un proceso dentro del container es comprometido.

## Modelo mental

Pensa las capabilities como una lista granular de permisos especificos ("puede tocar la luz", "puede tocar el agua") en vez del interruptor unico de "es dueño de la casa o no". Seccomp es una lista adicional de acciones fisicas completamente prohibidas dentro de la casa, sin importar que permisos tenga la persona, reduciendo lo que es posible hacer incluso con acceso legitimo.

## Ejemplo base

```bash
# Quitar todas las capabilities y agregar solo la estrictamente necesaria
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE mi-imagen

# Usar un perfil seccomp custom mas restrictivo que el default
docker run --security-opt seccomp=perfil-custom.json mi-imagen

# Verificar las capabilities activas de un container corriendo
docker inspect --format '{{.HostConfig.CapAdd}} {{.HostConfig.CapDrop}}' mi-container
```

## Como se usa en un proyecto real

1. Se aplica `--cap-drop=ALL` como punto de partida, agregando explicitamente (`--cap-add`) solo las capabilities que la aplicacion realmente necesita para funcionar.
2. Se evita `--privileged` (que otorga practicamente todos los privilegios y desactiva estas protecciones) salvo casos muy especificos y bien justificados (por ejemplo, herramientas de infraestructura que necesitan acceso profundo al host).
3. Se confia en el perfil seccomp default de Docker para la gran mayoria de aplicaciones, reservando perfiles custom mas restrictivos para servicios de alta sensibilidad.
4. Se documenta y justifica cualquier capability agregada explicitamente, como excepcion revisada en vez de configuracion por defecto.
5. Se integra la verificacion de "sin `--privileged` y con capabilities minimas" como parte de las revisiones de seguridad de despliegues en produccion.

## Criterio junior

Un junior debe entender conceptualmente que son las capabilities y seccomp, y saber que `--privileged` desactiva buena parte de estas protecciones.

## Criterio mid

Un mid debe aplicar `--cap-drop=ALL` con `--cap-add` selectivo para servicios de produccion, identificando exactamente que capabilities necesita cada aplicacion.

## Criterio senior

Un senior debe definir el estandar de hardening de containers para todo el equipo (capabilities minimas, seccomp, prohibicion de `--privileged` salvo excepcion justificada) y auditar que los despliegues de produccion cumplan ese estandar.

## Errores comunes

- Usar `--privileged` por comodidad para resolver un problema puntual, sin entender que desactiva gran parte del modelo de seguridad de Docker.
- No revisar que capabilities realmente necesita una aplicacion, dejando el set default de Docker sin evaluar si se puede restringir mas.
- Asumir que las capabilities y seccomp hacen innecesario correr como no-root (ver tema anterior), cuando en realidad son capas de defensa complementarias, no sustitutas.
- No documentar por que se agrego una capability especifica, generando configuraciones dificiles de auditar despues.

## Practica

- [ ] Corre un container con `--cap-drop=ALL` y verifica que funcionalidades de la aplicacion dejan de funcionar (si alguna).
- [ ] Agrega de vuelta solo la capability minima necesaria para que vuelva a funcionar.
- [ ] Investiga que hace el perfil seccomp default de Docker y que syscalls bloquea.
- [ ] Identifica si algun servicio de tu infraestructura usa `--privileged` y evalua si realmente lo necesita.
- [ ] Documenta las capabilities minimas necesarias para una aplicacion real que conozcas.

## Preguntas de repaso

- Que ventaja de seguridad dan las capabilities granulares sobre el modelo tradicional de "root o no root"?
- Que hace seccomp de forma distinta y complementaria a las capabilities?
- Por que `--privileged` es una bandera de alto riesgo que deberia evitarse salvo excepcion justificada?
- Por que las capabilities minimas no reemplazan la necesidad de correr como usuario no-root?
