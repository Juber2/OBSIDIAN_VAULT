---
tags:
  - NestJS
  - TypeScript
  - Nest-Core
Creado: 2026-07-27
Relacionado:
  - NestJS Core
  - NESTJS
---

# 01 - Modulos y Bootstrap

## Que vas a aprender

En este capitulo vas a entender como Nest arranca una aplicacion: el arbol de **modulos**, el proceso de **bootstrap** en `main.ts`, y como `imports`, `controllers`, `providers` y `exports` definen los limites de cada modulo.

## Explicacion

Un modulo Nest (`@Module()`) es la unidad basica de organizacion: agrupa controllers y providers relacionados con un dominio o feature. Toda aplicacion Nest tiene un `AppModule` raiz, que importa modulos de feature (`UsersModule`, `OrdersModule`, etc.) y modulos de infraestructura (`ConfigModule`, `TypeOrmModule`).

El bootstrap (`NestFactory.create(AppModule)`) recorre ese arbol, resuelve todas las dependencias declaradas y construye el contenedor de inyeccion de dependencias antes de levantar el servidor HTTP. Si algo en el grafo de dependencias esta mal (un provider no exportado, una dependencia circular), falla en este paso, no en runtime de un request.

## Modelo mental

Pensa cada modulo como una "caja" con una pared: adentro puede tener providers privados que solo el mismo modulo usa, y una puerta (`exports`) por donde deja salir lo que otros modulos pueden importar y usar. Sin `exports`, nada sale de la caja aunque este declarado en `providers`.

## Ejemplo base

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // visible para otros modulos que importen UsersModule
})
export class UsersModule {}

@Module({
  imports: [UsersModule, OrdersModule, ConfigModule.forRoot()],
})
export class AppModule {}
```

## Como se usa en un proyecto real

1. Cada dominio/feature tiene su propio modulo con controllers, providers y DTOs propios.
2. Los modulos de infraestructura (config, DB, logging) se importan como `forRoot()`/`forRootAsync()` en el `AppModule`.
3. Solo se exporta lo que otros modulos realmente necesitan consumir.
4. Se evita que dos modulos de feature se importen mutuamente (dependencia circular); si es necesario, se usa `forwardRef()` como ultimo recurso.
5. El `main.ts` configura pipes, filtros y prefijos globales antes de `app.listen()`.

## Criterio junior

Un junior debe poder crear un modulo, registrar controllers y providers, e importar un modulo en otro para reusar un servicio exportado.

## Criterio mid

Un mid debe poder diagnosticar errores de DI ("Nest can't resolve dependencies") revisando `imports`/`exports`, y evitar dependencias circulares con buen diseño en vez de `forwardRef()`.

## Criterio senior

Un senior debe diseñar el arbol de modulos completo de una aplicacion grande, decidir que es modulo de dominio vs modulo de infraestructura, y anticipar como ese arbol facilitaria una futura separacion en microservicios.

## Errores comunes

- Olvidar `exports` y no entender por que otro modulo no puede inyectar un servicio.
- Crear dependencias circulares entre modulos de feature por mal diseño de limites.
- Poner logica de negocio dentro del modulo raiz (`AppModule`) en vez de modulos de dominio.
- Reimportar el mismo modulo de infraestructura (`TypeOrmModule.forRoot()`) en varios lugares en vez de centralizarlo.

## Practica

- [ ] Crea dos modulos de dominio y haz que uno consuma un servicio exportado del otro.
- [ ] Provoca a proposito un error de dependencia no exportada y observa el mensaje.
- [ ] Crea una dependencia circular entre dos modulos y resuelvela con mejor diseño (no con `forwardRef`).
- [ ] Dibuja el arbol de modulos de un proyecto real que conozcas.
- [ ] Anota una regla de equipo sobre que puede ir en `AppModule`.

## Preguntas de repaso

- Que diferencia hay entre `providers` y `exports` en un modulo?
- Que pasa si intentas inyectar un servicio no exportado?
- Cuando es aceptable usar `forwardRef()`?
- Por que el bootstrap falla antes de levantar el servidor si hay un error de DI?
