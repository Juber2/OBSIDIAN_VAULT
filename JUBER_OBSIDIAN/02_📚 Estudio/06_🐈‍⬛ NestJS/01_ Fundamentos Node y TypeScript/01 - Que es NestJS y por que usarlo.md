---
tags:
  - NestJS
  - TypeScript
  - Nest-Fundamentos
Creado: 2026-07-27
Relacionado:
  - Fundamentos Node y TypeScript
  - NESTJS
---

# 01 - Que es NestJS y por que usarlo

## Que vas a aprender

En este capitulo vas a entender **que es NestJS**, que problema resuelve sobre Node.js puro o sobre Express/Fastify, y por que se convirtio en el framework backend TypeScript mas usado en equipos medianos y grandes.

## Explicacion

NestJS es un framework para construir aplicaciones backend en Node.js usando TypeScript de forma nativa. Toma ideas de Angular (modulos, decoradores, inyeccion de dependencias) y las aplica al backend, sobre un runtime HTTP que por defecto es Express (aunque puede correr sobre Fastify para mas performance).

La razon de su exito es que Node.js con Express da libertad total pero ninguna estructura: cada equipo termina inventando su propia arquitectura, con resultados muy dispares. NestJS impone una arquitectura opinada (modulos, controllers, providers, DI) que escala bien en equipos, sin perder el ecosistema de Node.

## Modelo mental

Pensalo como "Angular para el backend": una aplicacion Nest es un arbol de modulos, cada modulo agrupa controllers (entrada HTTP) y providers (logica y datos), y todo se conecta por inyeccion de dependencias. El framework no reemplaza a Node, es una capa de organizacion sobre Express/Fastify.

## Ejemplo base

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

## Como se usa en un proyecto real

1. Se instala con `npx @nestjs/cli new nombre-proyecto`.
2. Se define el arbol de modulos segun el dominio (users, auth, orders, etc.).
3. Cada modulo expone controllers y providers propios.
4. La configuracion de infraestructura (DB, env, guards globales) vive en el `AppModule` raiz.
5. Se decide desde el inicio si el runtime HTTP sera Express (default, mas ecosistema) o Fastify (mas velocidad).

## Criterio junior

Un junior debe poder explicar la diferencia entre Node.js, Express y NestJS, instalar el CLI, crear un proyecto y levantar un endpoint basico.

## Criterio mid

Un mid debe poder justificar cuando conviene NestJS frente a Express plano o Fastify plano, y reconocer el costo de adoptar un framework opinado (curva de aprendizaje, boilerplate, acoplamiento a decoradores).

## Criterio senior

Un senior debe poder decidir si NestJS es la eleccion correcta para un proyecto dado (monolito modular, microservicios, BFF, API publica) y documentar esa decision con tradeoffs claros para el equipo.

## Errores comunes

- Tratar NestJS como "solo Express con decoradores" y no aprovechar DI ni modularidad.
- Elegir NestJS para un script chico o una funcion serverless donde el boilerplate no aporta valor.
- No entender que Nest sigue corriendo sobre Node.js, con las mismas limitaciones de un solo hilo para CPU-bound.

## Practica

- [ ] Instala el Nest CLI y crea un proyecto nuevo.
- [ ] Levanta el servidor y confirma el endpoint por defecto.
- [ ] Compara el mismo endpoint hecho en Express puro.
- [ ] Anota tres diferencias estructurales entre ambos enfoques.
- [ ] Escribe una decision propia: cuando usarias NestJS y cuando no.

## Preguntas de repaso

- Que problema de Express/Node puro resuelve NestJS?
- Sobre que runtime HTTP corre NestJS por defecto?
- Que ventajas y costos trae adoptar un framework opinado?
- En que escenario NO elegirias NestJS?
