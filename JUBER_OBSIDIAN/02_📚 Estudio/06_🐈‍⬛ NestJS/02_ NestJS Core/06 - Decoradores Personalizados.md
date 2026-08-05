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

# 06 - Decoradores Personalizados

## Que vas a aprender

En este capitulo vas a aprender a crear **decoradores personalizados** en NestJS: decoradores de parametro (`createParamDecorator`), decoradores de metadata para guards (`SetMetadata`, `Reflector`), y como componer varios decoradores en uno solo.

## Explicacion

Nest permite crear decoradores propios para eliminar repeticion y expresar intencion de negocio directamente en la firma de un metodo. El caso mas comun es un decorador de parametro que extrae el usuario autenticado del request (`@CurrentUser()`), evitando repetir `req.user` en cada controller.

Otro patron clave es usar `SetMetadata()` para anotar un endpoint con informacion que despues un guard lee con `Reflector` (por ejemplo, `@Roles('admin')` que un `RolesGuard` interpreta para decidir acceso). Esto separa "que rol requiere este endpoint" (declarativo, en el controller) de "como se verifica eso" (logica, en el guard).

## Modelo mental

Pensa un decorador personalizado como una etiqueta legible que le pegas a un metodo o parametro, y que despues otra pieza del sistema (un guard, un pipe) sabe leer e interpretar. El decorador no ejecuta la logica, solo declara intencion.

## Ejemplo base

```typescript
export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const required = this.reflector.get<string[]>(ROLES_KEY, context.getHandler());
    if (!required) return true;
    const { user } = context.switchToHttp().getRequest();
    return required.includes(user?.role);
  }
}

@Roles('admin')
@Get('reportes')
getReportes(@CurrentUser() user: User) {
  return this.reportesService.findAll(user);
}
```

## Como se usa en un proyecto real

1. Se crea `@CurrentUser()` para evitar acceder a `req.user` manualmente en cada controller.
2. Se crean decoradores de metadata (`@Roles()`, `@Public()`) para expresar reglas de acceso de forma declarativa.
3. El guard correspondiente lee esa metadata con `Reflector` y aplica la logica real.
4. Se documentan los decoradores custom del proyecto en un lugar centralizado (`common/decorators`).
5. Se evita crear un decorador nuevo por cada variacion minima; se prioriza reusar y parametrizar.

## Criterio junior

Un junior debe poder usar decoradores personalizados ya creados por el equipo (`@CurrentUser()`, `@Roles()`) sin necesitar entender la implementacion interna en detalle.

## Criterio mid

Un mid debe poder crear un decorador de parametro simple y un decorador de metadata combinado con un guard que lo interprete.

## Criterio senior

Un senior debe diseñar el set de decoradores custom de la aplicacion como parte del lenguaje del dominio del equipo, manteniendo consistencia y evitando que se multipliquen decoradores redundantes o poco documentados.

## Errores comunes

- Repetir `request.user` o logica de extraccion de contexto en cada controller en vez de centralizarla en un decorador.
- Crear un decorador de metadata sin el guard que lo interprete, dejando la anotacion sin efecto real.
- No documentar que hace cada decorador custom, generando "magia" dificil de entender para nuevos integrantes.
- Sobre-abstraer con decoradores para casos que se usan una sola vez.

## Practica

- [ ] Crea un decorador `@CurrentUser()` que extraiga el usuario del request.
- [ ] Crea un decorador de metadata `@Roles()` junto a su guard correspondiente.
- [ ] Refactoriza un controller que accede a `req.user` manualmente para usar el nuevo decorador.
- [ ] Documenta los decoradores custom del proyecto en una carpeta `common/decorators`.
- [ ] Anota un caso donde NO valdria la pena crear un decorador nuevo.

## Preguntas de repaso

- Que hace `createParamDecorator` y en que se diferencia de un decorador de clase?
- Para que sirve `SetMetadata` combinado con `Reflector`?
- Por que separar "declarar que rol requiere un endpoint" de "verificar ese rol" es una buena practica?
- Que riesgo trae crear demasiados decoradores personalizados sin documentacion?
