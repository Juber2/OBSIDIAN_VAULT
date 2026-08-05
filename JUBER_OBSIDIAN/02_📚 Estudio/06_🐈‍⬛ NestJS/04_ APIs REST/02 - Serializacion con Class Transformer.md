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

# 02 - Serializacion con Class Transformer

## Que vas a aprender

En este capitulo vas a aprender a controlar exactamente que datos salen de tu API usando `class-transformer`: `@Exclude()`, `@Expose()`, el `ClassSerializerInterceptor`, y grupos de serializacion segun el rol del usuario que consulta.

## Explicacion

Por defecto, si devolvés una entidad completa desde un controller, Nest la serializa "tal cual", incluyendo campos que nunca deberian salir (password, tokens internos, campos de auditoria). `class-transformer` resuelve esto con `@Exclude()` sobre la entidad o DTO de respuesta, combinado con el `ClassSerializerInterceptor` que aplica esas reglas automaticamente a la respuesta HTTP.

Para casos mas finos, `@Expose({ groups: ['admin'] })` permite mostrar un campo solo a ciertos consumidores (por ejemplo, un admin ve el email completo de otro usuario, pero un usuario normal no).

## Modelo mental

Pensa la serializacion como un filtro de salida: la entidad interna puede tener todos los datos que necesita el sistema, pero antes de salir por HTTP pasa por un colador que solo deja pasar lo que esta explicitamente permitido para ese contexto.

## Ejemplo base

```typescript
@Entity()
export class User {
  @Column()
  email: string;

  @Exclude()
  @Column()
  password: string;
}

@Controller('users')
@UseInterceptors(ClassSerializerInterceptor)
export class UsersController {
  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id); // password nunca llega al cliente
  }
}
```

## Como se usa en un proyecto real

1. Se marca con `@Exclude()` cualquier campo sensible directamente en la entidad o en un DTO de respuesta dedicado.
2. Se activa `ClassSerializerInterceptor` de forma global en `main.ts` para no depender de que cada controller lo recuerde.
3. Se usan grupos (`groups`) cuando distintos roles necesitan ver distintos subconjuntos del mismo recurso.
4. Se prefiere un DTO de respuesta explicito sobre depender solo de excluir campos de la entidad, para dejar el contrato de salida totalmente claro.
5. Se testean los endpoints de lectura verificando que campos sensibles nunca aparezcan en la respuesta.

## Criterio junior

Un junior debe poder excluir un campo sensible de una entidad y activar el `ClassSerializerInterceptor` para que la exclusion se aplique automaticamente.

## Criterio mid

Un mid debe usar grupos de serializacion para exponer distintos subconjuntos de datos segun el rol del consumidor, y crear DTOs de respuesta explicitos para endpoints publicos sensibles.

## Criterio senior

Un senior debe auditar que ningun endpoint de la API exponga datos sensibles por omision, definir la politica de "que sale por default" (deny by default vs allow by default), y revisar en code review cualquier endpoint nuevo que devuelva una entidad sin pasar por serializacion controlada.

## Errores comunes

- Devolver una entidad completa sin `@Exclude()` en el campo de password u otros datos sensibles.
- Olvidar activar `ClassSerializerInterceptor`, dejando que `@Exclude()` no tenga ningun efecto.
- Confiar en la exclusion de campos de la entidad en vez de tener un DTO de respuesta explicito para contratos publicos criticos.
- No testear que la serializacion realmente oculta lo que se espera, descubriendo el problema en produccion.

## Practica

- [ ] Excluye un campo sensible de una entidad con `@Exclude()`.
- [ ] Activa `ClassSerializerInterceptor` de forma global.
- [ ] Crea un DTO de respuesta con grupos para admin y usuario normal.
- [ ] Escribe un test que verifique que un campo sensible nunca aparece en la respuesta.
- [ ] Audita un endpoint existente que devuelva una entidad completa sin control.

## Preguntas de repaso

- Que pasa si `@Exclude()` esta en la entidad pero no se activo `ClassSerializerInterceptor`?
- Para que sirven los grupos de serializacion?
- Por que un DTO de respuesta explicito es mas seguro que depender solo de excluir campos?
- Como testearias que un endpoint nunca expone un campo sensible?
