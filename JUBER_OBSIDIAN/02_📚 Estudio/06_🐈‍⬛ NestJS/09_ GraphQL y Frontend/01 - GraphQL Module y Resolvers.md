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

# 01 - GraphQL Module y Resolvers

## Que vas a aprender

En este capitulo vas a aprender a exponer una API GraphQL con `@nestjs/graphql`: **resolvers** como equivalente de los controllers REST, y como Nest integra Apollo Server por debajo.

## Explicacion

GraphQL invierte el control respecto a REST: en vez de que el servidor defina endpoints fijos con forma de respuesta fija, el cliente pide exactamente los campos que necesita en una unica request, incluso combinando datos de varias "entidades" relacionadas en una sola consulta. Un **resolver** en Nest es el equivalente a un controller: define como resolver cada campo de una query o mutation (`@Query()`, `@Mutation()`, `@ResolveField()`).

Nest integra Apollo Server (o Mercurius) por debajo, generando el schema GraphQL automaticamente a partir de los decoradores en el codigo TypeScript (**code first**) o leyendo un archivo `.graphql` (**schema first**, ver siguiente tema).

## Modelo mental

Pensa GraphQL como un menu a la carta en vez de un menu fijo: el cliente arma su propio pedido eligiendo exactamente que campos de que "platos" relacionados quiere, y el resolver de cada campo sabe como cocinar esa parte especifica, sin que el cliente tenga que hacer varias idas y vueltas al restaurante.

## Ejemplo base

```typescript
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  email: string;
}

@Resolver(() => User)
export class UsersResolver {
  constructor(private usersService: UsersService) {}

  @Query(() => [User])
  users() {
    return this.usersService.findAll();
  }

  @Mutation(() => User)
  createUser(@Args('dto') dto: CreateUserInput) {
    return this.usersService.create(dto);
  }
}
```

## Como se usa en un proyecto real

1. Se define el modulo GraphQL en `AppModule` con `GraphQLModule.forRoot()`, generalmente en modo code-first para mantener un unico lugar de verdad (el codigo TypeScript).
2. Cada resolver delega la logica real a un servicio, igual que un controller REST; el resolver es solo la capa de adaptacion al protocolo GraphQL.
3. Se usan `@ResolveField()` para resolver relaciones bajo demanda, con cuidado de no introducir N+1 (ver siguiente punto de performance con DataLoader).
4. Se combinan guards, interceptors y pipes igual que en REST, ya que Nest los aplica de forma transversal al mecanismo de transporte.
5. Se decide GraphQL para clientes con necesidades de datos muy variables (apps moviles con ancho de banda limitado, dashboards complejos) y REST para APIs mas simples o publicas.

## Criterio junior

Un junior debe poder definir un tipo GraphQL simple y un resolver con una query y una mutation basicas.

## Criterio mid

Un mid debe integrar resolvers con servicios existentes, resolver relaciones con `@ResolveField()`, y aplicar guards de autenticacion a queries y mutations.

## Criterio senior

Un senior debe decidir cuando GraphQL aporta valor real sobre REST para un proyecto dado, diseñar el schema pensando en evolucion sin breaking changes, y anticipar problemas de performance especificos de GraphQL (queries anidadas costosas, N+1).

## Errores comunes

- Poner logica de negocio directamente en el resolver en vez de delegar a un servicio, repitiendo el error de "controller gordo" visto en REST.
- No limitar la profundidad o complejidad de las queries permitidas, dejando que un cliente pida una consulta extremadamente costosa.
- No aplicar guards de autenticacion/autorizacion a mutations sensibles, asumiendo erroneamente que GraphQL es "mas seguro" por diseño.
- Diseñar el schema sin pensar en evolucion, forzando breaking changes tempranos.

## Practica

- [ ] Define un tipo GraphQL y un resolver con query y mutation basicas.
- [ ] Aplica un guard de autenticacion a una mutation sensible.
- [ ] Resuelve una relacion con `@ResolveField()`.
- [ ] Investiga como limitar la profundidad maxima de una query GraphQL.
- [ ] Anota cuando elegirias GraphQL sobre REST para un proyecto nuevo.

## Preguntas de repaso

- Que ventaja tiene GraphQL sobre REST para un cliente con necesidades de datos muy variables?
- Que responsabilidad tiene un resolver y en que se parece a un controller?
- Que riesgo tiene no limitar la complejidad de las queries permitidas?
- Por que los guards de Nest se aplican igual en GraphQL que en REST?
