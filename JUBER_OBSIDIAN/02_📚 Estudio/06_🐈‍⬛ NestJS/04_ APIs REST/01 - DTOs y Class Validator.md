---
tags:
  - NestJS
  - TypeScript
  - Nest-APIs
Creado: 2026-07-27
Relacionado:
  - APIs REST NestJS
  - NESTJS
---

# 01 - DTOs y Class Validator

## Que vas a aprender

En este capitulo vas a profundizar en el diseño de **DTOs** (Data Transfer Objects) como contrato explicito de entrada y salida de una API NestJS, y en el uso avanzado de `class-validator` para reglas de negocio simples a nivel de formato.

## Explicacion

Un DTO define exactamente que forma tiene el dato que entra o sale de un endpoint, independiente de como se modela internamente en la entidad de base de datos. Esta separacion es clave: la entidad puede cambiar (agregar una columna interna, un campo de auditoria) sin romper el contrato publico de la API, si el DTO se mantiene estable.

`class-validator` permite reglas declarativas (`@IsEmail`, `@IsEnum`, `@ValidateNested`, `@ArrayMinSize`) y reglas custom con `@ValidatorConstraint` para logica de validacion que no viene predefinida. Combinado con `class-transformer` (`@Type()`), tambien permite validar objetos anidados y arrays de objetos correctamente.

## Modelo mental

Pensa el DTO como el formulario oficial que un cliente de la API debe llenar: define exactamente que campos existen, cuales son obligatorios, y que formato deben tener, sin exponer nada de como se guarda esa informacion por dentro.

## Ejemplo base

```typescript
export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;
}

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsEnum(['admin', 'user'])
  role: string;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}
```

## Como se usa en un proyecto real

1. Se crea un DTO por operacion (`CreateUserDto`, `UpdateUserDto`, `UserResponseDto`), no uno solo reutilizado para todo.
2. Los DTOs anidados usan `@ValidateNested()` junto con `@Type()` para que la validacion realmente descienda al objeto interno.
3. Las reglas de negocio complejas (que no son solo formato) se validan en el servicio, no se fuerzan dentro del DTO.
4. Se documentan los DTOs con decoradores de Swagger (`@ApiProperty()`) para que la documentacion de API quede sincronizada con la validacion real.
5. Se versionan los DTOs cuando un contrato de API cambia de forma incompatible.

## Criterio junior

Un junior debe poder crear DTOs por operacion con reglas basicas de `class-validator` y entender por que no se reutiliza el mismo DTO para create y para la respuesta.

## Criterio mid

Un mid debe validar objetos anidados y arrays correctamente, y crear validadores custom para reglas de formato especificas del dominio.

## Criterio senior

Un senior debe definir la convencion de DTOs de todo el proyecto (naming, ubicacion, relacion con Swagger) y decidir la estrategia de versionado de contratos cuando la API evoluciona con clientes externos ya integrados.

## Errores comunes

- Reutilizar la misma clase para create, update y response, generando campos opcionales confusos y validaciones incorrectas.
- Olvidar `@Type()` en un DTO anidado y que `class-validator` no valide el objeto interno.
- Meter reglas de negocio complejas (que dependen de la base de datos) dentro de un validador de DTO.
- No mantener los DTOs sincronizados con la documentacion de Swagger, generando documentacion desactualizada.

## Practica

- [ ] Crea DTOs separados para create, update y response de un mismo recurso.
- [ ] Agrega un DTO anidado y valida que `@ValidateNested` + `@Type()` funcione correctamente.
- [ ] Crea un validador custom para una regla especifica de tu dominio.
- [ ] Documenta un DTO con decoradores de Swagger.
- [ ] Anota una convencion propia de naming de DTOs para tu equipo.

## Preguntas de repaso

- Por que conviene tener DTOs separados para create, update y response?
- Que pasa si falta `@Type()` en un DTO con un objeto anidado?
- Donde deberia vivir una regla de negocio que depende de consultar la base de datos?
- Como mantendrias sincronizada la documentacion Swagger con los DTOs reales?
