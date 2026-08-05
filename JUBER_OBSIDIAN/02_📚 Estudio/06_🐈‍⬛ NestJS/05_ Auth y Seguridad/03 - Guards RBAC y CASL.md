---
tags:
  - NestJS
  - TypeScript
  - Nest-Auth
Creado: 2026-07-27
Relacionado:
  - Auth y Seguridad NestJS
  - NESTJS
---

# 03 - Guards RBAC y CASL

## Que vas a aprender

En este capitulo vas a aprender a implementar **autorizacion**: control de acceso basado en roles (RBAC) con guards y decoradores, y control de acceso basado en permisos mas granular con la libreria **CASL**.

## Explicacion

RBAC (Role-Based Access Control) es el modelo mas simple: cada usuario tiene uno o mas roles (`admin`, `editor`, `user`), y cada endpoint declara que roles puede acceder. Se implementa con un `RolesGuard` que lee metadata puesta por un decorador `@Roles()` (visto en el capitulo de decoradores personalizados).

Cuando RBAC no alcanza (por ejemplo, "un editor solo puede editar sus propios posts, no los de otros"), se necesita autorizacion basada en **permisos y condiciones**. CASL permite definir reglas declarativas del tipo "puede `update` en `Post` si `post.authorId === user.id`", centralizando toda la logica de "quien puede hacer que" en un solo lugar (`abilities`).

## Modelo mental

RBAC responde "que rol tenes", CASL responde "que podes hacer especificamente sobre este recurso concreto". RBAC es una lista de nombres en la puerta; CASL es un guardia que revisa las condiciones especificas de cada caso antes de dejarte pasar.

## Ejemplo base

```typescript
// RBAC simple
@Roles('admin')
@UseGuards(JwtAuthGuard, RolesGuard)
@Delete(':id')
remove(@Param('id') id: string) {}

// CASL: permisos condicionales
export class AbilityFactory {
  createForUser(user: User) {
    const { can, build } = new AbilityBuilder(createMongoAbility);
    if (user.role === 'admin') can('manage', 'all');
    else can('update', 'Post', { authorId: user.id });
    return build();
  }
}

// En el servicio
const ability = this.abilityFactory.createForUser(user);
if (!ability.can('update', subject('Post', post))) throw new ForbiddenException();
```

## Como se usa en un proyecto real

1. RBAC cubre el caso comun: endpoints administrativos, features solo para ciertos roles.
2. CASL se introduce cuando aparece la necesidad de "es dueño del recurso" o reglas condicionales por atributo.
3. Las reglas de autorizacion se centralizan en un solo lugar (`AbilityFactory` o equivalente), nunca dispersas en ifs dentro de cada servicio.
4. Se testean las reglas de autorizacion de forma aislada, con distintos roles y distintos dueños de recurso.
5. Se documenta la matriz de permisos del sistema para que el equipo la entienda sin leer el codigo.

## Criterio junior

Un junior debe poder implementar RBAC simple con `@Roles()` y un `RolesGuard`, aplicandolo a endpoints administrativos.

## Criterio mid

Un mid debe reconocer cuando RBAC no alcanza (reglas condicionales por dueño de recurso) e introducir CASL o un patron equivalente para esos casos.

## Criterio senior

Un senior debe diseñar el modelo de autorizacion completo del sistema, decidir si conviene RBAC simple, RBAC+ownership, o un motor de reglas tipo CASL/ABAC, y mantener esa logica auditable y centralizada a medida que el numero de reglas crece.

## Errores comunes

- Repetir `if (user.role !== 'admin')` disperso en controllers y servicios en vez de centralizar en guards/abilities.
- Confundir "es dueño del recurso" con un rol, forzando roles artificiales para casos que en realidad son de ownership.
- No testear la matriz de permisos, dejando pasar combinaciones de rol/recurso no contempladas.
- Dejar que la logica de autorizacion crezca sin documentacion, volviendose dificil de auditar en seguridad.

## Practica

- [ ] Implementa RBAC simple con `@Roles()` y `RolesGuard` para un endpoint administrativo.
- [ ] Identifica un caso de tu dominio donde RBAC no alcanza (ownership).
- [ ] Implementa esa regla condicional con CASL o un patron equivalente propio.
- [ ] Escribe tests de autorizacion para distintos roles y distintos dueños.
- [ ] Documenta la matriz de permisos de tu sistema en una tabla simple.

## Preguntas de repaso

- Que diferencia hay entre RBAC y un modelo de permisos condicionales tipo CASL?
- Por que centralizar la logica de autorizacion facilita las auditorias de seguridad?
- Que caso de tu dominio no se resuelve bien solo con roles?
- Como testearias que un editor no puede modificar el post de otro usuario?
