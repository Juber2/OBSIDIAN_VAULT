---
tags:
  - NestJS
  - TypeScript
  - Nest-GraphQL
Creado: 2026-07-27
Relacionado:
  - GraphQL y Frontend
  - NESTJS
---

# 02 - Schema First vs Code First

## Que vas a aprender

En este capitulo vas a entender las dos formas de definir un schema GraphQL en NestJS: **code first** (el schema se genera desde decoradores TypeScript) y **schema first** (el schema se escribe en `.graphql` y Nest genera las interfaces TypeScript), y cuando conviene cada una.

## Explicacion

En **code first**, escribis clases TypeScript decoradas (`@ObjectType()`, `@Field()`) y Nest genera el archivo `.graphql` automaticamente. La ventaja es una unica fuente de verdad (el codigo), con el tipado de TypeScript siempre sincronizado con el schema, ideal para equipos backend-first.

En **schema first**, escribis el archivo `.graphql` a mano como contrato explicito, y Nest genera interfaces TypeScript a partir de el (`typings` autogenerados). La ventaja es que el schema es legible y editable por cualquiera (incluyendo el equipo frontend) sin tocar codigo TypeScript, y sirve como documentacion de contrato explicita e independiente del lenguaje del backend.

## Modelo mental

Pensa code first como escribir el codigo y dejar que el "menu" (schema) se imprima solo a partir de el. Pensa schema first como escribir el "menu" primero, como contrato acordado con todos los interesados, y despues generar el codigo que lo implementa.

## Ejemplo base

```typescript
// Code first
@ObjectType()
export class Product {
  @Field(() => ID) id: string;
  @Field() name: string;
  @Field(() => Float) price: number;
}
```

```graphql
# Schema first: product.graphql
type Product {
  id: ID!
  name: String!
  price: Float!
}

type Query {
  products: [Product!]!
}
```

## Como se usa en un proyecto real

1. Se elige code first cuando el equipo es principalmente backend TypeScript y se prioriza tipado sincronizado automaticamente.
2. Se elige schema first cuando el contrato GraphQL debe negociarse explicitamente con otros equipos (frontend, equipos externos) antes de implementar, o en proyectos poliglotas.
3. Se evita mezclar ambos enfoques en el mismo proyecto sin una razon clara, para no confundir de donde "nace" la verdad del schema.
4. En code first, se documentan los tipos con `description` en los decoradores para que el schema generado siga siendo legible.
5. Se versiona el archivo `.graphql` generado (o escrito a mano) en el repositorio para revisar cambios de contrato en cada PR.

## Criterio junior

Un junior debe poder definir un tipo simple en code first y entender que existe la alternativa de schema first.

## Criterio mid

Un mid debe elegir el enfoque correcto segun el contexto del equipo y mantener el schema (generado o escrito) legible y bien documentado.

## Criterio senior

Un senior debe definir el enfoque estandar del equipo para todos los modulos GraphQL, considerando quien consume el contrato y como se coordina su evolucion entre equipos.

## Errores comunes

- Mezclar code first y schema first en el mismo proyecto sin una razon documentada, generando confusion sobre la fuente de verdad.
- No documentar los tipos en code first, generando un schema tecnicamente correcto pero dificil de leer para el equipo frontend.
- No revisar el diff del schema generado en cada PR, dejando pasar breaking changes de contrato sin darse cuenta.
- Elegir schema first sin necesidad real de coordinacion externa, sumando el paso extra de generacion de tipos sin beneficio proporcional.

## Practica

- [ ] Define el mismo tipo con ambos enfoques (code first y schema first) para comparar la experiencia.
- [ ] Agrega descripciones a los campos en code first para mejorar la legibilidad del schema generado.
- [ ] Revisa el diff del schema generado en un cambio de ejemplo.
- [ ] Anota que enfoque elegirias para tu equipo actual y por que.

## Preguntas de repaso

- Cual es la fuente de verdad del schema en code first, y cual en schema first?
- Cuando conviene schema first sobre code first?
- Que riesgo tiene mezclar ambos enfoques en el mismo proyecto?
- Como asegurarias que un cambio de schema no rompa a los consumidores existentes?
