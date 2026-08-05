---
tags:
  - Docker
  - DevOps
  - Docker-Monitoreo
Creado: 2026-07-28
Relacionado:
  - Monitoreo y Logging
  - DOCKER
---

# 01 - docker logs y Drivers de Logging

## Que vas a aprender

En este capitulo vas a profundizar en como Docker maneja los logs de un container: el driver de logging por defecto (`json-file`), sus limitaciones, y las alternativas disponibles para produccion.

## Explicacion

Docker captura automaticamente todo lo que un container escribe en **stdout** y **stderr**, sin necesitar configuracion adicional en la aplicacion (por eso las aplicaciones dentro de containers deben loggear ahi, nunca a un archivo interno que se pierde con el container). El driver por defecto, `json-file`, guarda esos logs como archivos JSON en el disco del host, accesibles con `docker logs`.

El problema del driver `json-file` sin configuracion adicional es que **crece sin limite**: sin `max-size` y `max-file` configurados, los logs de un container de larga duracion pueden llenar el disco del servidor. Para produccion real, ademas es comun usar drivers que envian los logs directamente a un sistema centralizado (`syslog`, `fluentd`, `awslogs`), en vez de depender solo de leer archivos locales del host.

## Modelo mental

Pensa el driver de logging como la decision de donde el container "escribe su diario": por defecto, lo escribe en un cuaderno local que crece sin control si nadie le pone un limite de paginas. Enviar logs a un sistema centralizado es como que el diario se escriba directamente en una biblioteca compartida, organizada y con capacidad de busqueda, en vez de en cuadernos sueltos dispersos por cada servidor.

## Ejemplo base

```bash
docker run -d --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 mi-imagen
# Limita cada archivo de log a 10MB, con maximo 3 archivos rotados (30MB total)

docker logs -f --tail 100 mi-container   # sigue el log, mostrando las ultimas 100 lineas iniciales
```

```yaml
# docker-compose.yml con limite de logs configurado
services:
  api:
    image: mi-imagen
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

## Como se usa en un proyecto real

1. Se configura `max-size` y `max-file` en todo servicio de produccion, evitando que los logs llenen el disco del servidor con el tiempo.
2. Se envian logs a un sistema centralizado (ver siguiente tema) para aplicaciones criticas, en vez de depender solo del almacenamiento local del host.
3. Se recuerda que la aplicacion dentro del container debe loggear a stdout/stderr, nunca a un archivo interno del filesystem del container que se pierde al recrearlo.
4. Se usa `docker logs --since` y `--until` para acotar el rango temporal de logs revisados durante un diagnostico especifico.
5. Se documenta la politica de retencion de logs local del equipo, coordinada con la politica de logs centralizados si existe.

## Criterio junior

Un junior debe entender que la aplicacion debe loggear a stdout/stderr para que Docker los capture, y poder usar `docker logs` con las opciones basicas.

## Criterio mid

Un mid debe configurar `max-size`/`max-file` en todos los servicios de produccion para evitar que los logs llenen el disco sin control.

## Criterio senior

Un senior debe diseñar la estrategia de logging de toda la infraestructura, decidiendo entre drivers locales con limites vs envio directo a un sistema centralizado, segun la criticidad de cada servicio.

## Errores comunes

- No configurar `max-size`/`max-file`, dejando que los logs de un container de larga duracion llenen el disco del servidor sin limite.
- Loggear a un archivo dentro del filesystem del container en vez de a stdout/stderr, perdiendo esos logs cuando el container se recrea.
- Depender solo de logs locales del host sin ningun sistema centralizado, complicando el diagnostico en sistemas con multiples servidores.
- No revisar periodicamente el uso de disco relacionado a logs en servidores de produccion.

## Practica

- [ ] Configura `max-size` y `max-file` para un servicio de un proyecto de practica.
- [ ] Verifica que la aplicacion loggea correctamente a stdout/stderr y que `docker logs` la captura.
- [ ] Usa `docker logs --since` para acotar un rango temporal especifico de busqueda.
- [ ] Investiga que drivers de logging soporta tu version de Docker para enviar logs a un sistema centralizado.
- [ ] Documenta la politica de retencion de logs local que aplicarias en produccion.

## Preguntas de repaso

- Por que una aplicacion dentro de un container deberia loggear a stdout/stderr en vez de a un archivo interno?
- Que problema resuelve configurar `max-size` y `max-file` en el driver de logging?
- Que ventaja da enviar logs a un sistema centralizado sobre depender solo de logs locales del host?
- Como acotarias la busqueda de logs a un rango temporal especifico durante un diagnostico?
