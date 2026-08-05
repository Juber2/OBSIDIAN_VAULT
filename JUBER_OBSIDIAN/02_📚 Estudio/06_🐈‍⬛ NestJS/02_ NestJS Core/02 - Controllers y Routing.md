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

# 02 - Controllers y Routing

## Que vas a aprender

En este capitulo vas a dominar **controllers**: como se definen rutas HTTP con decoradores, como se leen parametros, query strings, body y headers, y como versionar y prefijar rutas.

## Explicacion

Un controller (`@Controller('users')`) agrupa endpoints relacionados bajo un prefijo comun. Cada metodo se mapea a un verbo HTTP con decoradores (`@Get`, `@Post`, `@Patch`, `@Delete`), y los datos de entrada se extraen con decoradores de parametro (`@Param`, `@Query`, `@Body`, `@Headers`).

Los controllers deben ser **delgados**: su unica responsabilidad es traducir HTTP a llamadas de servicio y traducir el resultado de vuelta a una respuesta. Toda logica de negocio, validacion compleja o acceso a datos vive en providers (servicios), no en el controller.

## Modelo mental

Pensa el controller como un traductor en la frontera: recibe un idioma (HTTP: headers, body, query) y lo convierte al idioma interno de la aplicacion (llamadas a metodos tipados), y despues traduce la respuesta de vuelta. Un traductor no deberia tomar decisiones de negocio.

## Ejemplo base

```typescript
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll(@Query('page') page = 1) {
    return this.usersService.findAll(Number(page));
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}
```

## Como se usa en un proyecto real

1. Cada controller representa un recurso o subrecurso claro (`/users`, `/users/:id/orders`).
2. Los parametros se tipan y validan con pipes (`ParseIntPipe`, `ValidationPipe`) en vez de castear a mano.
3. El controller nunca accede directo a la base de datos; siempre pasa por un provider.
4. Se define una convencion de codigos de estado HTTP (`@HttpCode`) coherente para todo el equipo.
5. Se versiona la API (`/v1/users`) desde el inicio si se espera evolucion de contrato.

## Criterio junior

Un junior debe poder crear un controller con endpoints CRUD basicos usando los decoradores correctos para cada tipo de dato de entrada.

## Criterio mid

Un mid debe mantener controllers delgados, delegar toda logica a servicios, y manejar correctamente codigos de estado y parametros opcionales.

## Criterio senior

Un senior debe definir convenciones de API para el equipo (nombres de recursos, versionado, codigos de estado, estructura de respuestas) y detectar en code review cuando un controller esta acumulando logica que no le corresponde.

## Errores comunes

- Meter logica de negocio o acceso a base de datos directo en el controller.
- No validar ni tipar parametros de ruta o query, dejando pasar strings donde se esperan numeros.
- Devolver estructuras de respuesta inconsistentes entre distintos endpoints.
- No versionar la API y romper clientes existentes al cambiar un contrato.

## Practica

- [ ] Crea un controller CRUD completo para un recurso simple.
- [ ] Aplica pipes de parseo a los parametros de ruta.
- [ ] Revisa un controller existente y mueve logica de negocio a un servicio.
- [ ] Define una convencion de respuesta estandar (envelope) para tu API.
- [ ] Anota como versionarias la API si tuvieras que cambiar un contrato existente.

## Preguntas de repaso

- Que responsabilidad deberia tener (y no tener) un controller?
- Que decorador usarias para leer un parametro de ruta tipado como numero?
- Como versionarias un endpoint sin romper clientes existentes?
- Por que es mala practica acceder a la base de datos directo desde el controller?
