---
tags:
  - NestJS
  - TypeScript
  - Nest-Performance
Creado: 2026-07-27
Relacionado:
  - Performance NestJS
  - NESTJS
---

# 04 - Escalabilidad y Clustering

## Que vas a aprender

En este capitulo vas a aprender las estrategias para escalar una aplicacion NestJS bajo carga creciente: **escalado vertical vs horizontal**, **clustering** con multiples procesos Node en la misma maquina, y los requisitos para que una aplicacion sea realmente **stateless** y escalable horizontalmente.

## Explicacion

El escalado vertical (mas CPU/RAM en el mismo servidor) tiene un techo fisico y de costo. El escalado horizontal (mas instancias del mismo servicio, detras de un load balancer) es la estrategia que realmente escala, pero exige que la aplicacion sea **stateless**: ninguna instancia deberia guardar estado que otra instancia no pueda ver (sesiones en memoria local, cache en memoria local, locks en memoria local son todos anti-patrones ya vistos en capitulos anteriores).

Dentro de una misma maquina, como Node.js usa un solo hilo por proceso, se puede aprovechar mejor el hardware con **clustering** (modulo nativo `cluster` de Node, o el modo `-i max` de PM2), que levanta un proceso Node por core de CPU disponible, todos escuchando en el mismo puerto a traves de un balanceador interno.

## Modelo mental

Pensa el escalado vertical como agrandar un solo restaurante (mas mesas, mas cocina), con un limite fisico de cuanto podes agrandarlo. Pensa el escalado horizontal como abrir sucursales identicas del mismo restaurante: cada una atiende una parte del publico, y podes abrir tantas como necesites, siempre que ninguna sucursal dependa de informacion que solo tiene ella (todo el estado compartido vive en un lugar comun, como Redis o la base de datos).

## Ejemplo base

```typescript
// Clustering nativo simplificado
import cluster from 'node:cluster';
import { cpus } from 'node:os';

if (cluster.isPrimary) {
  cpus().forEach(() => cluster.fork());
} else {
  bootstrap(); // cada worker corre su propia instancia de Nest
}
```

```bash
# Equivalente con PM2, mas simple de operar
pm2 start dist/main.js -i max
```

## Como se usa en un proyecto real

1. Se verifica que la aplicacion sea stateless antes de escalar horizontalmente: sesiones en Redis, cache en Redis, sin locks ni contadores en memoria local (ver capitulos anteriores).
2. Se usa clustering (PM2 o nativo) para aprovechar todos los cores de cada maquina antes de sumar mas maquinas.
3. Se configura un load balancer (o el balanceador del proveedor cloud) delante de las instancias, con health checks para sacar de rotacion instancias no saludables.
4. Se definen metricas de auto-escalado (CPU, memoria, latencia, largo de cola) para que la infraestructura sume o quite instancias segun demanda real.
5. Se prueban las instancias bajo carga (load testing) antes de asumir que escalan como se espera.

## Criterio junior

Un junior debe entender la diferencia entre escalado vertical y horizontal, y por que Node.js se beneficia de clustering para aprovechar multiples cores.

## Criterio mid

Un mid debe identificar y eliminar estado local que impida escalar horizontalmente (sesiones, cache, locks en memoria), y configurar clustering basico con PM2.

## Criterio senior

Un senior debe diseñar la estrategia de escalado completa del sistema (vertical vs horizontal, auto-escalado, particionamiento de base de datos si aplica) y validar con load testing real que el sistema efectivamente escala como se espera antes de que ocurra bajo trafico real inesperado.

## Errores comunes

- Asumir que agregar instancias resuelve un cuello de botella que en realidad esta en la base de datos compartida (el cuello de botella se mueve, no desaparece).
- Mantener estado en memoria local de una instancia, rompiendo el comportamiento cuando el load balancer distribuye requests entre instancias distintas.
- No probar el sistema bajo carga real antes de asumir que escala, descubriendo limites solo durante un pico real de trafico.
- Escalar horizontalmente sin health checks apropiados, dejando trafico llegando a instancias no saludables.

## Practica

- [ ] Audita un proyecto Nest existente en busca de estado en memoria local que impida escalar horizontalmente.
- [ ] Configura clustering con PM2 y verifica que se usen todos los cores disponibles.
- [ ] Diseña la configuracion de health checks para un load balancer delante de multiples instancias.
- [ ] Investiga como hacer un load test basico contra tu API.
- [ ] Anota el cuello de botella mas probable de tu sistema al escalar horizontalmente (probablemente no sea la aplicacion en si).

## Preguntas de repaso

- Por que una aplicacion debe ser stateless para escalar horizontalmente de forma confiable?
- Que ventaja da el clustering dentro de una misma maquina antes de sumar mas maquinas?
- Que pasa si agregas mas instancias de aplicacion pero el cuello de botella real esta en la base de datos?
- Por que es importante hacer load testing antes de asumir que un sistema escala?
