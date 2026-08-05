---
tags:
  - Docker
  - DevOps
  - Docker-Senior
Creado: 2026-07-28
Relacionado:
  - Senior Docker
  - DOCKER
---

# 04 - Costos y Performance en Produccion

## Que vas a aprender

En este capitulo vas a aprender a gestionar **limites de recursos** de containers en produccion (CPU y memoria), y como esas decisiones impactan tanto en performance como en costo real de infraestructura.

## Explicacion

Sin limites explicitos, un container puede consumir todos los recursos disponibles del host, afectando a otros containers vecinos (un problema conocido como "noisy neighbor"). Docker permite definir **limites** (`--memory`, `--cpus`) y, en algunos contextos, tambien **reservas** (garantia minima). Un limite de memoria mal dimensionado tiene dos riesgos opuestos: demasiado bajo causa que el container sea matado por el kernel (OOM killed) bajo carga normal; demasiado alto desperdicia capacidad que podria usarse para correr mas containers en el mismo servidor, incrementando el costo de infraestructura sin necesidad real.

El dimensionamiento correcto se basa en **datos reales de consumo** (ver capitulo de Monitoreo, metricas historicas), no en suposiciones. Ademas, el costo de infraestructura de containers esta directamente relacionado con la densidad lograda: imagenes mas chicas (capitulo de Optimizacion) y limites bien ajustados permiten correr mas servicios por servidor, reduciendo el costo total.

## Modelo mental

Pensa los limites de recursos como el presupuesto de espacio asignado a cada inquilino de un edificio: si le das demasiado poco espacio a alguien, se ahoga (OOM killed) en cuanto necesita mas de lo normal; si le das demasiado a todos "por las dudas", el edificio entero cabe muchos menos inquilinos de los que podria (menor densidad, mayor costo por inquilino).

## Ejemplo base

```yaml
services:
  api:
    image: mi-registro/api:1.4.0
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 256M
```

```bash
docker run -d --memory=512m --memory-swap=512m --cpus=1 mi-imagen
# --memory-swap igual a --memory desactiva el uso de swap, evitando degradacion silenciosa de performance
```

## Como se usa en un proyecto real

1. Se dimensionan limites basados en metricas historicas reales de consumo (capitulo de Monitoreo), no en valores arbitrarios "que suenan razonables".
2. Se define tanto limite (maximo permitido) como reserva (garantia minima) cuando el orquestador lo soporta, para un dimensionamiento mas preciso.
3. Se monitorea la tasa de eventos OOM killed como señal directa de que los limites de memoria estan mal dimensionados (demasiado bajos).
4. Se revisa periodicamente la densidad de containers por servidor como metrica de eficiencia de costos de infraestructura.
5. Se ajustan los limites en un ciclo continuo a medida que la aplicacion evoluciona y su patron de consumo real cambia con el tiempo.

## Criterio junior

Un junior debe poder definir limites basicos de CPU y memoria para un container, y entender que un limite muy bajo puede causar que el container sea matado (OOM killed).

## Criterio mid

Un mid debe dimensionar limites basados en metricas reales de consumo, y diagnosticar eventos OOM killed como señal de mal dimensionamiento.

## Criterio senior

Un senior debe optimizar la densidad de containers por servidor como estrategia directa de reduccion de costos de infraestructura, balanceando esa densidad contra el riesgo de "noisy neighbors" y la necesidad de margen para picos de trafico.

## Errores comunes

- No definir ningun limite de recursos, permitiendo que un container consuma todos los recursos disponibles y afecte a sus vecinos.
- Definir limites de memoria demasiado ajustados sin datos reales, causando eventos OOM killed frecuentes bajo carga normal.
- Definir limites demasiado generosos "por las dudas", desperdiciando capacidad y aumentando el costo de infraestructura sin necesidad real.
- No revisar y ajustar los limites periodicamente a medida que el patron de consumo real de la aplicacion cambia con el tiempo.

## Practica

- [ ] Define limites de CPU y memoria para un servicio basandote en metricas historicas reales de consumo.
- [ ] Provoca un evento OOM killed a proposito con un limite demasiado bajo y observa el comportamiento.
- [ ] Calcula la densidad de containers que podrias correr en un servidor con distintos escenarios de dimensionamiento.
- [ ] Investiga como monitorearias la tasa de eventos OOM killed en tu infraestructura.
- [ ] Documenta el proceso de revision periodica de limites que aplicarias a medida que un servicio evoluciona.

## Preguntas de repaso

- Que riesgo opuesto tienen un limite de memoria demasiado bajo y uno demasiado alto?
- Que es un evento "OOM killed" y que señala sobre el dimensionamiento de un container?
- Como se relaciona la densidad de containers por servidor con el costo real de infraestructura?
- Por que el dimensionamiento de limites deberia basarse en datos historicos reales en vez de suposiciones?
