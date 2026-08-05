---
tags:
  - Linux
  - Performance
  - Linux-Performance
Creado: 2026-07-28
Relacionado:
  - Performance y Troubleshooting
  - LINUX
---

# 03 - Diagnostico de Problemas de Red

## Que vas a aprender

En este capitulo vas a aprender un flujo sistematico para diagnosticar problemas de conectividad de red, combinando herramientas ya vistas (`ping`, `dig`, `ss`) con **`tcpdump`** para inspeccion de paquetes de bajo nivel cuando el problema no es evidente con herramientas de mas alto nivel.

## Explicacion

Ante un problema de red, el flujo sistematico va de lo general a lo especifico: primero verificar conectividad basica a nivel IP (`ping`), despues verificar resolucion de nombres (`dig`, ya visto en el capitulo de Redes), despues verificar que el puerto especifico del servicio esta realmente accesible (`nc -zv host puerto` o `curl` para HTTP), y solo si el problema persiste sin explicacion clara, bajar al nivel de paquetes individuales con `tcpdump`, capturando el trafico real para ver exactamente que esta pasando (o no pasando) en la red.

`tcpdump` captura paquetes en una interfaz de red, mostrando el trafico real byte a byte (o resumido), permitiendo confirmar cosas como: si un paquete realmente sale del servidor, si llega una respuesta, o si un firewall esta silenciosamente descartando trafico sin ningun error visible del lado de la aplicacion. Es la herramienta de "ultimo recurso" cuando las capas superiores no dan suficiente informacion.

## Modelo mental

Pensa el diagnostico de red como investigar por que una carta nunca llego a destino: primero confirmas que la direccion existe (`ping`), despues que el nombre en el sobre se traduce correctamente a esa direccion (`dig`), despues que alguien en esa direccion especifica esta realmente esperando el correo (verificar el puerto), y si nada de eso explica el problema, revisas la camara de seguridad del correo (`tcpdump`) para ver exactamente que paso con la carta en cada punto de su recorrido.

## Ejemplo base

```bash
ping -c 4 servidor.com                    # conectividad IP basica
dig servidor.com                           # resolucion de nombres correcta

nc -zv servidor.com 443                     # verifica si el puerto 443 esta accesible
curl -v https://servidor.com                # request HTTP completo con detalle verboso

# tcpdump: capturar trafico especifico para inspeccionar en detalle
sudo tcpdump -i eth0 host servidor.com -w captura.pcap
sudo tcpdump -i eth0 port 443 -c 20         # 20 paquetes en el puerto 443
```

## Como se usa en un proyecto real

1. Se sigue el flujo sistematico (IP -> DNS -> puerto -> paquetes) en vez de saltar directamente a herramientas complejas sin haber descartado primero las causas mas simples.
2. Se usa `nc -zv` o `curl -v` para verificar si un servicio especifico esta realmente escuchando y respondiendo, distinto de solo verificar conectividad IP general.
3. Se usa `tcpdump` como ultimo recurso cuando el problema no se explica con herramientas de mas alto nivel, capturando evidencia concreta del trafico real.
4. Se guarda la captura de `tcpdump` (`-w archivo.pcap`) para analisis posterior mas detallado con herramientas graficas como Wireshark si es necesario.
5. Se documenta cada diagnostico complejo de red como caso de estudio, construyendo conocimiento reutilizable para el equipo ante problemas similares futuros.

## Criterio junior

Un junior debe poder seguir el flujo basico de diagnostico (`ping`, `dig`, verificar puerto) ante un problema simple de conectividad.

## Criterio mid

Un mid debe usar `curl -v` y `nc -zv` para diagnosticar problemas especificos de un servicio, distinguiendolos de problemas generales de red.

## Criterio senior

Un senior debe usar `tcpdump` para diagnosticar problemas complejos de red donde las capas superiores no dan suficiente informacion, y liderar la investigacion sistematica de incidentes de conectividad dificiles de reproducir.

## Errores comunes

- Saltar directamente a `tcpdump` sin haber descartado primero causas mas simples con herramientas de mas alto nivel, complicando innecesariamente el diagnostico.
- No verificar el puerto especifico de un servicio, confundiendo "el servidor responde ping" con "el servicio especifico esta funcionando".
- No guardar capturas de `tcpdump` para analisis posterior, perdiendo evidencia util si el problema requiere investigacion mas profunda.
- No documentar diagnosticos complejos de red, perdiendo ese conocimiento para incidentes similares en el futuro.

## Practica

- [ ] Sigue el flujo sistematico completo (ping, dig, verificar puerto) para diagnosticar la conectividad hacia un servicio real.
- [ ] Usa `curl -v` para ver el detalle completo de una request HTTP, incluyendo headers.
- [ ] Captura trafico especifico con `tcpdump` sobre un puerto de interes.
- [ ] Guarda una captura en formato `.pcap` para analisis posterior.
- [ ] Documenta un caso de diagnostico de red (real o simulado) como referencia para tu equipo.

## Preguntas de repaso

- Por que conviene seguir un flujo sistematico de lo general a lo especifico ante un problema de red?
- Que diferencia hay entre verificar conectividad IP con `ping` y verificar que un servicio especifico responde con `curl` o `nc`?
- Cuando se justifica usar `tcpdump` en vez de herramientas de mas alto nivel?
- Que ventaja da guardar una captura de `tcpdump` en vez de solo observarla en tiempo real?
