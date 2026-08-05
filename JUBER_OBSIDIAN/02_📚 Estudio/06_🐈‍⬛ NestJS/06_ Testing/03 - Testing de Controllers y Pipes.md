---
tags:
  - NestJS
  - TypeScript
  - Nest-Testing
Creado: 2026-07-27
Relacionado:
  - Testing NestJS
  - NESTJS
---

# 03 - Testing de Controllers y Pipes

## Que vas a aprender

En este capitulo vas a aprender a testear **controllers** de forma unitaria (mockeando el servicio que consumen), y a testear **pipes** y **guards** de forma aislada usando un `ExecutionContext` simulado.

## Explicacion

Un test de controller no deberia levantar toda la aplicacion HTTP: se construye un `TestingModule` con el controller real y el servicio mockeado, y se llama al metodo del controller directamente, verificando que delega correctamente al servicio y devuelve lo esperado. Esto es rapido y aislado, a diferencia de un test e2e que si levanta el servidor HTTP completo.

Para pipes, guards e interceptors, el reto es que reciben un `ExecutionContext` (o `ArgumentMetadata` para pipes) que en un test hay que construir manualmente o mockear, ya que no hay un request HTTP real corriendo.

## Modelo mental

Pensa el test de controller como verificar que el "traductor" (ver capitulo de Controllers) llama correctamente al interprete (servicio) y devuelve lo que el interprete le dio, sin preocuparte por como funciona el interprete por dentro, eso ya lo cubre el test del servicio.

## Ejemplo base

```typescript
describe('UsersController', () => {
  let controller: UsersController;
  let service: { findOne: jest.Mock };

  beforeEach(async () => {
    service = { findOne: jest.fn() };
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [{ provide: UsersService, useValue: service }],
    }).compile();
    controller = module.get(UsersController);
  });

  it('deberia devolver el usuario del servicio', async () => {
    service.findOne.mockResolvedValue({ id: '1', email: 'a@a.com' });
    const result = await controller.findOne('1');
    expect(result).toEqual({ id: '1', email: 'a@a.com' });
  });
});

// Test de guard
const mockContext = {
  switchToHttp: () => ({ getRequest: () => ({ headers: { authorization: 'Bearer x' } }) }),
} as unknown as ExecutionContext;

expect(new AuthGuard().canActivate(mockContext)).toBe(true);
```

## Como se usa en un proyecto real

1. Los tests de controller mockean el servicio y verifican delegacion correcta, no logica de negocio (esa ya la testea el servicio).
2. Los guards e interceptors se testean construyendo un `ExecutionContext` minimo con exactamente los datos que el guard necesita leer.
3. Los pipes se testean llamando directamente a `transform()` con distintos valores de entrada, incluyendo casos invalidos.
4. Se evita duplicar en el test de controller la misma logica ya cubierta en el test de servicio.
5. Se documenta con un comentario minimo cuando el mock de `ExecutionContext` es deliberadamente parcial (solo lo que se necesita).

## Criterio junior

Un junior debe poder testear un controller simple mockeando el servicio que usa, y verificar que devuelve el resultado esperado.

## Criterio mid

Un mid debe testear guards y pipes de forma aislada, construyendo el contexto minimo necesario, y evitar duplicar cobertura entre tests de controller y de servicio.

## Criterio senior

Un senior debe definir donde termina la responsabilidad de un test de controller y donde empieza la de un test e2e, evitando duplicacion de esfuerzo de testing entre distintas capas de la piramide.

## Errores comunes

- Testear logica de negocio completa dentro del test de controller, duplicando lo que ya deberia estar cubierto en el test del servicio.
- Construir un `ExecutionContext` mock incompleto que hace que el guard falle por razones ajenas a lo que se queria testear.
- No testear pipes con casos invalidos, dejando sin cobertura el camino de rechazo.
- Mezclar test de controller con llamadas HTTP reales, perdiendo la velocidad y aislamiento de un test unitario.

## Practica

- [ ] Escribe un test de controller que mockee el servicio y verifique delegacion correcta.
- [ ] Escribe un test de guard con un `ExecutionContext` mockeado minimo.
- [ ] Escribe un test de pipe que verifique tanto un valor valido como uno invalido.
- [ ] Revisa un test de controller existente y elimina logica de negocio duplicada del servicio.
- [ ] Anota la diferencia de alcance entre un test de controller y un test e2e para tu equipo.

## Preguntas de repaso

- Que deberia verificar un test de controller que un test de servicio no cubre?
- Como se construye un `ExecutionContext` minimo para testear un guard?
- Por que testear un pipe con un valor invalido es tan importante como testear uno valido?
- Donde esta el limite entre lo que cubre un test unitario de controller y un test e2e?
