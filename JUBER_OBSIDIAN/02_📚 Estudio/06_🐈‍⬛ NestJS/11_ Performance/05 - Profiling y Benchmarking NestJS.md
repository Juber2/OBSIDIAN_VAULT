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

# 05 - Profiling y Benchmarking NestJS

## Que vas a aprender

En este capitulo vas a aprender a medir performance de forma rigurosa en vez de optimizar por intuicion: **profiling** de CPU y memoria de un proceso Node, y **benchmarking** de endpoints con herramientas de carga como k6 o autocannon.

## Explicacion

Optimizar sin medir primero es apostar: es comun "optimizar" una parte del codigo que en realidad no era el cuello de botella real, mientras el problema verdadero sigue intacto en otro lugar. El **profiling** (con el inspector nativo de Node, `--prof`, o herramientas como Clinic.js) muestra exactamente donde el proceso gasta tiempo de CPU o donde crece el uso de memoria, identificando el cuello de botella real con datos, no con intuicion.

El **benchmarking** de endpoints (con `autocannon` para pruebas simples, `k6` para escenarios mas realistas con multiples tipos de request) mide throughput y latencia bajo carga simulada, permitiendo comparar el impacto real de un cambio (antes/despues) y detectar regresiones de performance antes de que lleguen a produccion.

## Modelo mental

Pensa el profiling como una radiografia del proceso: en vez de adivinar que hueso esta roto, ves exactamente donde esta el problema. El benchmarking es someter al paciente a un esfuerzo controlado y medir como responde, para saber si esta realmente en condiciones de aguantar la carga real que le espera.

## Ejemplo base

```bash
# Profiling con Clinic.js
npx clinic doctor -- node dist/main.js
# hace requests mientras corre, genera un reporte visual de donde se gasta el tiempo

# Benchmark simple con autocannon
npx autocannon -c 50 -d 30 http://localhost:3000/productos
# 50 conexiones concurrentes durante 30 segundos, reporta throughput y latencia
```

```javascript
// Escenario mas realista con k6
import http from 'k6/http';
export default function () {
  http.get('http://localhost:3000/productos');
}
export const options = { vus: 100, duration: '1m' };
```

## Como se usa en un proyecto real

1. Se establece una linea base de performance (latencia p50/p95/p99, throughput) antes de cualquier optimizacion, para poder medir el impacto real de un cambio.
2. Se usa profiling cuando se sospecha un cuello de botella de CPU o una fuga de memoria, en vez de optimizar codigo al azar.
3. Se corren benchmarks de carga en un ambiente similar a produccion, no en la laptop del desarrollador, para que los numeros sean representativos.
4. Se integra un benchmark basico en el pipeline de CI para detectar regresiones de performance antes de mergear cambios criticos.
5. Se documentan los resultados de cada ronda de optimizacion, para construir un historial de decisiones basadas en datos.

## Criterio junior

Un junior debe poder correr un benchmark basico con autocannon contra un endpoint y leer sus resultados de latencia y throughput.

## Criterio mid

Un mid debe usar profiling para identificar un cuello de botella real antes de optimizar, y comparar metricas antes/despues de un cambio con benchmarks reproducibles.

## Criterio senior

Un senior debe establecer el proceso de performance testing del equipo (linea base, umbrales aceptables, integracion en CI), y liderar decisiones de optimizacion basadas en datos medidos, no en intuicion o preferencias personales.

## Errores comunes

- Optimizar codigo basandose en intuicion sin haber hecho profiling, gastando tiempo en algo que no era el cuello de botella real.
- Correr benchmarks en un entorno no representativo (laptop del desarrollador) y sacar conclusiones erroneas sobre performance en produccion.
- No establecer una linea base antes de optimizar, haciendo imposible medir si un cambio realmente mejoro algo.
- No repetir benchmarks despues de cambios importantes, perdiendo visibilidad sobre regresiones de performance con el tiempo.

## Practica

- [ ] Corre un benchmark basico con autocannon contra un endpoint de un proyecto de practica.
- [ ] Usa Clinic.js (o herramienta equivalente) para hacer profiling de ese mismo endpoint.
- [ ] Aplica una optimizacion basada en lo que encontraste en el profiling, no en intuicion.
- [ ] Compara el benchmark antes y despues de la optimizacion.
- [ ] Anota como integrarias un benchmark basico en tu pipeline de CI.

## Preguntas de repaso

- Por que optimizar sin haber hecho profiling primero es arriesgado?
- Que diferencia hay entre profiling y benchmarking?
- Por que un benchmark corrido en la laptop del desarrollador puede no ser representativo?
- Que ventaja da tener una linea base de performance antes de empezar a optimizar?
