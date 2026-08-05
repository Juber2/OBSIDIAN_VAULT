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

# 02 - Unit Testing de Providers y Servicios

## Que vas a aprender

En este capitulo vas a aprender a testear **servicios** de forma unitaria, aislando sus dependencias con mocks a traves del contenedor de DI de Nest, usando `overrideProvider()`.

## Explicacion

Testear un servicio de forma unitaria significa probar su logica en aislamiento, sin tocar base de datos real ni servicios externos reales. Nest facilita esto porque, al construir el `TestingModule`, se puede reemplazar cualquier provider real por un mock con `overrideProvider(Token).useValue(mock)`.

Esto es especialmente poderoso combinado con repositorios de TypeORM o Prisma: en vez de conectarse a una base de datos real, se inyecta un objeto mock que simula los metodos usados (`find`, `save`) y se verifica que el servicio los llame con los argumentos correctos.

## Modelo mental

Pensa el test unitario de un servicio como poner esa pieza en una mesa de pruebas aislada, con actores de utileria en lugar de sus dependencias reales: podes verificar exactamente como reacciona el servicio ante distintas respuestas de esas dependencias, sin depender de una base de datos real ni de servicios externos.

## Ejemplo base

```typescript
describe('UsersService', () => {
  let service: UsersService;
  let repo: jest.Mocked<Repository<User>>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: { findOne: jest.fn(), save: jest.fn() },
        },
      ],
    }).compile();

    service = module.get(UsersService);
    repo = module.get(getRepositoryToken(User));
  });

  it('deberia lanzar NotFoundException si el usuario no existe', async () => {
    repo.findOne.mockResolvedValue(null);
    await expect(service.findOne('id-inexistente')).rejects.toThrow(NotFoundException);
  });
});
```

## Como se usa en un proyecto real

1. Se mockean todas las dependencias externas (repositorios, servicios HTTP, colas) al testear un servicio de forma unitaria.
2. Se testean tanto el camino feliz como los casos de error (excepciones esperadas, validaciones que fallan).
3. Se verifica no solo el resultado sino, cuando es relevante, que se llamo a la dependencia correcta con los argumentos correctos (`toHaveBeenCalledWith`).
4. Se evita mockear demasiado profundo (mockear detalles internos de una libreria) y se prefiere mockear en el limite del servicio.
5. Se mantiene cada test enfocado en un solo comportamiento, con nombres descriptivos de que se espera.

## Criterio junior

Un junior debe poder mockear una dependencia simple con `useValue` y testear el camino feliz de un metodo de servicio.

## Criterio mid

Un mid debe testear tanto casos exitosos como de error, verificar interacciones con mocks (`toHaveBeenCalledWith`), y mantener los mocks minimos y claros.

## Criterio senior

Un senior debe definir que nivel de mockeo es apropiado para cada tipo de dependencia, evitar tests fragiles por sobre-mockeo, y asegurar que la suite de tests unitarios realmente de confianza para refactorizar sin miedo.

## Errores comunes

- No testear los casos de error (excepciones, validaciones fallidas), dejando esos caminos sin cobertura.
- Mockear tan profundo que el test termina verificando la implementacion del mock en vez del comportamiento del servicio.
- Compartir mocks mutables entre tests sin resetearlos, generando tests que pasan o fallan segun el orden de ejecucion.
- Escribir un test gigante que verifica demasiadas cosas a la vez, dificultando saber que fallo exactamente.

## Practica

- [ ] Escribe un test unitario que mockee un repositorio y testee el camino feliz.
- [ ] Escribe un test que verifique que el servicio lanza la excepcion correcta ante un caso de error.
- [ ] Verifica con `toHaveBeenCalledWith` que el servicio llama al mock con los argumentos correctos.
- [ ] Resetea los mocks entre tests con `jest.clearAllMocks()` en `afterEach`.
- [ ] Anota una regla propia sobre que tan profundo mockear tus dependencias.

## Preguntas de repaso

- Como reemplazarias un repositorio real por un mock en un `TestingModule`?
- Por que es importante testear tambien los casos de error, no solo el camino feliz?
- Que riesgo tiene mockear demasiado profundo una dependencia?
- Para que sirve `toHaveBeenCalledWith` en un test de servicio?
