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

# 05 - Firewall con iptables nftables

## Que vas a aprender

En este capitulo vas a aprender a configurar un firewall a nivel de sistema operativo: **iptables** (el clasico, todavia ampliamente usado), su sucesor moderno **nftables**, y **ufw** como interfaz simplificada sobre iptables para Ubuntu/Debian.

## Explicacion

`iptables` filtra paquetes de red segun **tablas**, **cadenas** y **reglas**: la cadena `INPUT` controla trafico entrante al servidor, `OUTPUT` el saliente, `FORWARD` el trafico que pasa a traves del servidor (relevante para routers o para Docker, que modifica reglas de iptables internamente para su networking). Cada regla especifica un criterio de coincidencia (puerto, protocolo, IP origen) y una accion (`ACCEPT`, `DROP`, `REJECT`).

`nftables` es el reemplazo moderno del kernel para `iptables`, con sintaxis mas limpia y mejor performance, aunque `iptables` sigue siendo ampliamente usado y comprendido (muchas distribuciones lo mantienen como capa de compatibilidad sobre `nftables`). **`ufw`** (Uncomplicated Firewall) es una interfaz simplificada sobre iptables, ya vista de forma practica en el capitulo de Fundamentos de este libro, ideal para casos simples sin necesitar dominar la sintaxis completa de reglas.

## Modelo mental

Pensa el firewall como el personal de seguridad en la entrada de un edificio: revisa cada persona (paquete) que intenta entrar o salir, comparandola contra una lista de reglas ("solo dejar pasar visitantes con cita en el piso 3", "rechazar a cualquiera del edificio de al lado"), decidiendo si la deja pasar, la rechaza silenciosamente, o le informa explicitamente que no puede entrar.

## Ejemplo base

```bash
# ufw: interfaz simple, ideal para la mayoria de casos
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp        # SSH
sudo ufw allow 80,443/tcp    # HTTP/HTTPS
sudo ufw enable

# iptables: control granular directo
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -j DROP    # rechaza todo lo demas por defecto
sudo iptables -L -v                # lista reglas actuales con contadores
```

## Como se usa en un proyecto real

1. Se usa `ufw` para la mayoria de servidores donde la simplicidad es prioridad sobre control granular extremo, siguiendo el patron ya visto en el capitulo de Fundamentos.
2. Se aplica una politica de **denegar por defecto** el trafico entrante, permitiendo explicitamente solo los puertos necesarios (SSH, HTTP/HTTPS, y cualquier puerto de aplicacion especifico).
3. Se usa `iptables -L -v` (o el equivalente de `nftables`) para auditar periodicamente que reglas estan activas y detectar reglas obsoletas o innecesariamente permisivas.
4. Se coordina la configuracion del firewall del host con las reglas que Docker modifica automaticamente (visto en el capitulo de Redes de Docker), evitando conflictos o sorpresas de exposicion no intencional.
5. Se documenta cada regla no obvia del firewall (por que se permite un puerto o IP especifica) para que futuros administradores entiendan el razonamiento.

## Criterio junior

Un junior debe poder configurar un firewall basico con `ufw`, permitiendo solo los puertos necesarios y denegando el resto.

## Criterio mid

Un mid debe leer y auditar reglas de `iptables` existentes, y diagnosticar cuando el firewall esta bloqueando trafico que deberia estar permitido.

## Criterio senior

Un senior debe diseñar la politica de firewall de toda la infraestructura, coordinando reglas de host con las que Docker/Kubernetes modifican automaticamente, y auditar periodicamente la superficie de exposicion real de cada servidor.

## Errores comunes

- No aplicar una politica de "denegar por defecto", dejando el firewall en un estado mas permisivo de lo necesario.
- Configurar reglas de firewall sin verificar que el puerto de SSH sigue permitido antes de aplicar la politica de denegacion, arriesgando quedar bloqueado del propio servidor.
- No auditar reglas existentes periodicamente, dejando reglas obsoletas o excesivamente permisivas acumuladas con el tiempo.
- No coordinar las reglas del firewall del host con las que Docker modifica automaticamente, generando comportamiento de exposicion inesperado.

## Practica

- [ ] Configura `ufw` con politica de denegar por defecto, permitiendo solo SSH y los puertos de aplicacion necesarios.
- [ ] Verifica que el puerto SSH sigue accesible antes de activar la politica de denegacion (para no quedar bloqueado).
- [ ] Lista y audita las reglas activas de `iptables` en un sistema con `iptables -L -v`.
- [ ] Investiga como Docker modifica automaticamente reglas de iptables para su networking.
- [ ] Documenta la politica de firewall que aplicarias a un servidor de produccion nuevo.

## Preguntas de repaso

- Que significa aplicar una politica de "denegar por defecto" en un firewall?
- Que cadena de iptables controla trafico entrante, y cual controla trafico que pasa a traves del servidor?
- Que precaucion tomarias antes de aplicar una politica de denegacion en un servidor remoto por SSH?
- Como se relaciona la configuracion del firewall del host con las reglas que Docker modifica automaticamente?
