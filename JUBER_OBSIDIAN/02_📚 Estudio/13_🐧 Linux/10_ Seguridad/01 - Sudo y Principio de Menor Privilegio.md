---
tags:
  - Linux
  - Seguridad
  - Linux-Seguridad
Creado: 2026-07-28
Relacionado:
  - Seguridad
  - LINUX
---

# 01 - Sudo y Principio de Menor Privilegio

## Que vas a aprender

En este capitulo vas a profundizar en **sudo**, el mecanismo estandar para elevar privilegios de forma controlada, y como configurarlo con reglas granulares que apliquen el principio de menor privilegio (ya visto de forma consistente en los libros de Docker y NestJS).

## Explicacion

`sudo` permite que un usuario normal ejecute comandos especificos con privilegios de root, quedando registrado en logs (`/var/log/auth.log` o equivalente) quien ejecuto que comando y cuando, a diferencia de trabajar directamente como root donde esa trazabilidad se pierde. El archivo `/etc/sudoers` (editado siempre con `visudo`, que valida sintaxis antes de guardar, evitando dejar el sistema en un estado roto) define quien puede usar sudo y con que alcance.

La configuracion granular permite ir mas alla de "todo o nada": se puede dar a un usuario permiso de sudo solo para comandos especificos (`usuario ALL=(ALL) /usr/bin/systemctl restart mi-servicio`), sin darle acceso completo de root, aplicando el principio de menor privilegio de forma mucho mas precisa que simplemente agregarlo al grupo `sudo`/`wheel` con acceso total.

## Modelo mental

Pensa `sudo` como una llave maestra temporal que se le entrega a un empleado solo para la tarea especifica que necesita hacer, quedando registrado exactamente cuando la uso y para que, en vez de darle una copia permanente de la llave maestra completa del edificio (trabajar como root directamente) sin ningun registro de que hizo con ella.

## Ejemplo base

```bash
sudo visudo                     # edita /etc/sudoers de forma segura, valida sintaxis

# En /etc/sudoers, dar acceso granular sin privilegios completos:
# deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart mi-api

sudo -l                          # lista que comandos puede ejecutar el usuario actual con sudo
sudo -u otro_usuario comando      # ejecuta como otro usuario especifico, no necesariamente root

grep sudo /var/log/auth.log       # audita el uso historico de sudo (Debian/Ubuntu)
```

## Como se usa en un proyecto real

1. Se otorga sudo granular para tareas administrativas especificas (reiniciar un servicio propio, ver logs de un servicio) en vez de acceso completo de root para usuarios que solo necesitan esa capacidad puntual.
2. Se usa siempre `visudo` para editar `/etc/sudoers`, nunca un editor de texto directo, evitando dejar el archivo con sintaxis invalida que podria bloquear el acceso de sudo para todos.
3. Se audita periodicamente `/var/log/auth.log` (o equivalente) para revisar el uso de sudo en servidores criticos, detectando actividad anomala.
4. Se evita trabajar directamente como root en el dia a dia, prefiriendo sudo para cada comando que realmente necesita privilegios elevados, manteniendo trazabilidad completa.
5. Se documenta claramente que reglas de sudoers existen y por que, para que auditorias de seguridad futuras puedan justificar cada permiso otorgado.

## Criterio junior

Un junior debe poder usar `sudo` para comandos que requieren privilegios, y entender por que es preferible a trabajar directamente como root.

## Criterio mid

Un mid debe configurar reglas granulares en `/etc/sudoers` con `visudo`, limitando el acceso de sudo a comandos especificos segun necesidad real.

## Criterio senior

Un senior debe auditar y mantener la politica de sudo de toda la infraestructura, revocando accesos innecesarios, y estableciendo el proceso de aprobacion para nuevas reglas de sudoers.

## Errores comunes

- Trabajar directamente como root en vez de usar sudo, perdiendo trazabilidad de que comandos se ejecutaron y quien los ejecuto.
- Editar `/etc/sudoers` con un editor de texto directo en vez de `visudo`, arriesgando dejar el archivo con sintaxis invalida.
- Dar acceso de sudo completo (`ALL=(ALL) ALL`) cuando el usuario solo necesita ejecutar un comando especifico repetidamente.
- No auditar el uso de sudo periodicamente, perdiendo visibilidad de actividad administrativa potencialmente anomala.

## Practica

- [ ] Verifica que comandos podes ejecutar con sudo actualmente usando `sudo -l`.
- [ ] Configura una regla granular en `/etc/sudoers` (con `visudo`) que otorgue permiso solo para un comando especifico.
- [ ] Revisa `/var/log/auth.log` (o equivalente) para ver el historial de uso de sudo en tu sistema.
- [ ] Compara el riesgo de trabajar como root directamente versus usar sudo para cada comando.
- [ ] Documenta una politica de sudo que aplicarias para distintos roles de tu equipo.

## Preguntas de repaso

- Que ventaja de trazabilidad da usar sudo sobre trabajar directamente como root?
- Por que se debe usar `visudo` en vez de un editor de texto directo para modificar `/etc/sudoers`?
- Como configurarias sudo para que un usuario solo pueda reiniciar un servicio especifico, sin acceso completo de root?
- Que informacion revisarias al auditar el uso historico de sudo en un servidor?
