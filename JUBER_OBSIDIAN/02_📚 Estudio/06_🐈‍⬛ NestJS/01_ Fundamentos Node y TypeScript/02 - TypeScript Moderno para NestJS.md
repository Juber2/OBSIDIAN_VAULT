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

# 02 - TypeScript Moderno para NestJS

## Que vas a aprender

En este capitulo vas a repasar el subconjunto de **TypeScript** que realmente se usa todos los dias en NestJS: decoradores, tipos, genericos, interfaces vs types, y la configuracion de `tsconfig.json` que Nest necesita para funcionar.

## Explicacion

NestJS depende fuertemente de **decoradores** (`@Controller`, `@Injectable`, `@Module`) y de **metadata reflejada en runtime** via `reflect-metadata`. Esto exige un `tsconfig.json` con `experimentalDecorators` y `emitDecoratorMetadata` activados. Sin entender esto, muchos errores de DI ("cannot resolve dependency") son dificiles de diagnosticar.

Ademas del uso de decoradores, el dia a dia en Nest usa: interfaces para contratos, DTOs con clases (no interfaces, porque `class-validator` necesita clases reales en runtime), genericos para repositorios y servicios reutilizables, y tipos utilitarios (`Partial`, `Omit`, `Pick`) para variantes de DTOs.

## Modelo mental

Los decoradores en Nest no son azucar sintactico: son la forma en que el framework construye, en runtime, un grafo de dependencias a partir de metadata. Cuando ves `@Injectable()` sobre una clase, pensa "esta clase queda registrada como candidata para ser inyectada en otras".

## Ejemplo base

```typescript
export class CreateUserDto {
  email: string;
  password: string;
}

export class UpdateUserDto implements Partial<CreateUserDto> {
  email?: string;
  password?: string;
}

@Injectable()
export class UsersService {
  create(dto: CreateUserDto): Promise<User> {
    // logica
    return Promise.resolve({} as User);
  }
}
```

## Como se usa en un proyecto real

1. Se configura `tsconfig.json` con `strict: true` para evitar bugs de tipado silenciosos.
2. Los DTOs se escriben como clases, nunca interfaces, para poder decorarlos con `class-validator`.
3. Se usan tipos utilitarios (`Partial`, `Omit`) para derivar DTOs de update a partir de DTOs de create.
4. Se evita `any`; cuando el tipo es realmente desconocido se usa `unknown` y se narrowea.
5. Se documentan los tipos de dominio (entidades, DTOs, value objects) separados de los tipos de infraestructura.

## Criterio junior

Un junior debe entender que son los decoradores, saber declarar una clase con tipos basicos y DTOs simples, y no confundir `interface` con `class` en el contexto de validacion.

## Criterio mid

Un mid debe usar genericos para servicios y repositorios reutilizables, tipos utilitarios para variantes de DTOs, y configurar `tsconfig.json` con reglas estrictas sin romper el proyecto.

## Criterio senior

Un senior debe definir convenciones de tipado para el equipo (cuando usar `type` vs `interface`, como versionar DTOs, como evitar duplicacion de tipos entre capas) y detectar en code review usos de `any` o `as` que esconden bugs.

## Errores comunes

- Definir DTOs como `interface` y descubrir en runtime que `class-validator` no valida nada.
- Usar `any` para "hacer andar" un error de tipos en vez de resolver la causa.
- No activar `strict` y arrastrar errores de null/undefined hasta produccion.
- Duplicar tipos entre la capa de dominio y la capa de persistencia sin necesidad.

## Practica

- [ ] Configura un `tsconfig.json` estricto en un proyecto Nest.
- [ ] Escribe un DTO de create y deriva uno de update con `Partial`.
- [ ] Crea un servicio generico reutilizable con un tipo generico.
- [ ] Reemplaza un `any` existente por un tipo correcto.
- [ ] Anota una convencion de tipado para tu equipo.

## Preguntas de repaso

- Por que los DTOs en Nest deben ser clases y no interfaces?
- Que hace `emitDecoratorMetadata` y por que Nest lo necesita?
- Cuando usarias `type` y cuando `interface`?
- Que riesgo trae `any` en un proyecto Nest grande?
