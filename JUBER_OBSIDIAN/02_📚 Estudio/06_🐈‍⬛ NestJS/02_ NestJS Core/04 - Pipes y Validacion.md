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

# 04 - Pipes y Validacion

## Que vas a aprender

En este capitulo vas a entender que son los **pipes** en NestJS: transformacion y validacion de datos de entrada, el `ValidationPipe` global con `class-validator`/`class-transformer`, y como crear pipes personalizados.

## Explicacion

Un pipe se ejecuta justo antes de que un handler de controller reciba sus parametros. Tiene dos usos: **transformar** datos (por ejemplo, convertir un string de query a numero con `ParseIntPipe`) y **validar** datos (rechazar un request si no cumple reglas, devolviendo 400 automaticamente).

El patron mas usado en produccion es un `ValidationPipe` global combinado con DTOs decorados con `class-validator` (`@IsEmail()`, `@IsString()`, `@MinLength()`). Esto centraliza toda la validacion de entrada sin escribir ifs manuales en cada controller.

## Modelo mental

Pensa los pipes como un control de aduana antes de que el dato entre al pais (al handler): revisan que el dato tenga la forma correcta, lo convierten al formato esperado, y si algo no cumple, lo rechazan antes de que cause problemas adentro.

## Ejemplo base

```typescript
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}

// main.ts
app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }));

// Pipe personalizado
@Injectable()
export class ParseObjectIdPipe implements PipeTransform {
  transform(value: string) {
    if (!isValidObjectId(value)) throw new BadRequestException('Id invalido');
    return value;
  }
}
```

## Como se usa en un proyecto real

1. Se registra `ValidationPipe` de forma global en `main.ts`, con `whitelist: true` para descartar campos no declarados en el DTO.
2. Cada endpoint de escritura tiene su propio DTO con reglas de `class-validator`.
3. Se crean pipes personalizados para casos especificos del dominio (validar un formato de ID propio, normalizar un valor).
4. Se separan mensajes de error de validacion para que el frontend pueda mostrarlos campo por campo.
5. Se testean los DTOs de validacion con casos limite (campos vacios, tipos incorrectos, valores fuera de rango).

## Criterio junior

Un junior debe poder crear un DTO con decoradores de `class-validator` y activar el `ValidationPipe` global.

## Criterio mid

Un mid debe crear pipes personalizados para casos de negocio especificos y configurar opciones avanzadas del `ValidationPipe` (`whitelist`, `transform`, `forbidNonWhitelisted`).

## Criterio senior

Un senior debe definir la estrategia de validacion de todo el proyecto (donde vive la validacion de formato vs la validacion de reglas de negocio) y asegurar que los mensajes de error sean utiles y consistentes para los consumidores de la API.

## Errores comunes

- No activar `whitelist`, permitiendo que el cliente envie campos extra que terminan persistidos sin control.
- Validar "a mano" con ifs dentro del controller en vez de usar `class-validator`.
- Confundir validacion de formato (DTO) con validacion de reglas de negocio (que pertenece al servicio/dominio).
- No testear los DTOs, dejando pasar bugs de validacion a produccion.

## Practica

- [ ] Crea un DTO con al menos cinco reglas distintas de `class-validator`.
- [ ] Activa el `ValidationPipe` global con `whitelist` y `forbidNonWhitelisted`.
- [ ] Crea un pipe personalizado para una regla especifica de tu dominio.
- [ ] Escribe un test que verifique que un DTO invalido es rechazado.
- [ ] Anota la diferencia entre una regla que va en el DTO y una que va en el servicio.

## Preguntas de repaso

- Que diferencia hay entre transformar y validar en el contexto de los pipes?
- Que hace la opcion `whitelist` del `ValidationPipe`?
- Cuando crearias un pipe personalizado en vez de usar uno de `class-validator`?
- Donde deberia vivir una regla de negocio compleja que no es simple validacion de formato?
