---
tags:
  - Linux
  - Redes
  - Linux-Redes
Creado: 2026-07-28
Relacionado:
  - Redes
  - LINUX
---

# 02 - Configuracion de Red con ip y nmcli

## Que vas a aprender

En este capitulo vas a aprender a configurar interfaces de red en Linux con el comando moderno **`ip`** (sucesor de `ifconfig`, obsoleto en distribuciones actuales), y con **NetworkManager** (`nmcli`) para gestion mas persistente en sistemas de escritorio y algunos servidores.

## Explicacion

El comando `ip` (parte del paquete `iproute2`) es el estandar moderno para configurar interfaces de red, direcciones, y rutas: `ip addr add` asigna una direccion IP a una interfaz, `ip link set` activa/desactiva interfaces, `ip route add` agrega rutas. Los cambios hechos con `ip` directamente son **temporales**: se pierden al reiniciar la interfaz o el sistema, a menos que se persistan en archivos de configuracion especificos de la distribucion (`/etc/netplan` en Ubuntu moderno, `/etc/network/interfaces` en Debian tradicional, `nmcli` con NetworkManager).

**NetworkManager** (controlado con `nmcli`) es el gestor de red mas usado en distribuciones de escritorio y muchos servidores modernos, dando persistencia automatica de la configuracion y manejo mas amigable de conexiones (WiFi, VPN, multiples interfaces), siendo la opcion recomendada cuando esta disponible sobre editar archivos de configuracion manualmente.

## Modelo mental

Pensa `ip` como hacer cambios directos y temporales en el tablero de control de la red (efectivos de inmediato, pero se resetean si alguien reinicia el sistema), y `nmcli`/NetworkManager como un asistente que ademas de hacer el cambio, lo anota en un registro permanente para que se aplique automaticamente cada vez que el sistema arranca.

## Ejemplo base

```bash
# Cambios temporales con ip (se pierden al reiniciar la interfaz)
ip addr add 192.168.1.100/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.1.1

# Gestion persistente con NetworkManager
nmcli connection show                      # lista conexiones configuradas
nmcli device status                         # estado de cada interfaz
nmcli connection modify eth0 ipv4.addresses 192.168.1.100/24
nmcli connection up eth0                    # aplica y persiste el cambio
```

## Como se usa en un proyecto real

1. Se usa `ip` para diagnostico y cambios temporales de prueba, verificando el efecto antes de persistir la configuracion definitivamente.
2. Se usa `nmcli` (donde NetworkManager esta disponible) para configuracion persistente en servidores que lo usan como gestor de red por defecto.
3. Se verifica que metodo de persistencia usa la distribucion especifica (netplan, interfaces, NetworkManager) antes de intentar hacer un cambio permanente, dado que varia entre distribuciones.
4. Se documenta la configuracion de red de servidores criticos, incluyendo el metodo usado para persistirla, como parte de la documentacion de infraestructura.
5. Se prueba cualquier cambio de red critico con acceso de respaldo disponible (como una consola fuera de banda), dado que un error de configuracion de red puede dejar un servidor remoto inaccesible.

## Criterio junior

Un junior debe poder ver el estado de las interfaces de red con `ip addr` y `nmcli device status`.

## Criterio mid

Un mid debe configurar una direccion IP y ruta con `ip`, y hacer un cambio persistente equivalente con `nmcli` o el metodo de persistencia de la distribucion.

## Criterio senior

Un senior debe estandarizar el metodo de configuracion de red usado en toda la infraestructura del equipo, documentando el proceso y previniendo cambios que puedan dejar servidores remotos inaccesibles.

## Errores comunes

- Hacer un cambio de red con `ip` esperando que persista, sorprendiendose cuando se pierde al reiniciar el sistema.
- No tener acceso de respaldo (consola fuera de banda) antes de hacer un cambio critico de red en un servidor remoto, arriesgando quedar sin acceso si algo sale mal.
- Mezclar metodos de configuracion (editar archivos manualmente y usar NetworkManager al mismo tiempo) generando conflictos de configuracion.
- No verificar que metodo de persistencia usa la distribucion especifica antes de intentar un cambio permanente.

## Practica

- [ ] Revisa el estado de tus interfaces de red con `ip addr` y (si esta disponible) `nmcli device status`.
- [ ] Haz un cambio temporal de configuracion con `ip` y verifica que se pierde al reiniciar la interfaz.
- [ ] Investiga que metodo de persistencia usa tu distribucion (netplan, interfaces, NetworkManager).
- [ ] Si tenes NetworkManager disponible, haz un cambio persistente con `nmcli`.
- [ ] Documenta el proceso de configuracion de red que usarias en un servidor de produccion.

## Preguntas de repaso

- Por que los cambios hechos con `ip` directamente son temporales?
- Que ventaja da NetworkManager (`nmcli`) sobre editar archivos de configuracion manualmente?
- Que precaucion tomarias antes de hacer un cambio critico de red en un servidor remoto?
- Como verificarias que metodo de persistencia de red usa una distribucion especifica?
