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

# 01 - Jest y Estructura de Tests

## Que vas a aprender

En este capitulo vas a repasar **Jest**, el test runner que el Nest CLI configura por defecto, y la convencion de organizacion de tests en un proyecto Nest: unitarios (`.spec.ts`) junto al codigo, y e2e (`.e2e-spec.ts`) en una carpeta separada.

## Explicacion

Jest provee todo lo necesario para testear un proyecto Nest sin herramientas adicionales: `describe`/`it`, matchers (`expect`), mocks (`jest.fn()`, `jest.mock()`), y un runner rapido con soporte de TypeScript via `ts-jest` o `swc`. El Nest CLI genera automaticamente un archivo `.spec.ts` junto a cada recurso creado con `nest generate`.

La convencion es que los tests unitarios vivan **junto al archivo que testean** (mismo directorio), facilitando encontrar el test de cualquier archivo, mientras que los tests end-to-end viven en una carpeta `test/` separada porque prueban la aplicacion completa levantada, no una unidad aislada.

## Modelo mental

Pensa cada test como una especificacion ejecutable: describe el comportamiento esperado de una pieza de codigo de forma que, si alguien rompe ese comportamiento sin darse cuenta, el test lo grita antes de que llegue a produccion.

## Ejemplo base

```typescript
describe('UsersService', () => {
  let service: UsersService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('deberia estar definido', () => {
    expect(service).toBeDefined();
  });
});
```

## Como se usa en un proyecto real

1. Cada servicio, controller y pipe importante tiene su archivo `.spec.ts` junto al codigo.
2. Se usa `Test.createTestingModule()` para construir un modulo de test aislado, con solo las dependencias necesarias.
3. Se define un umbral minimo de cobertura en CI (no como meta en si misma, sino como red de seguridad).
4. Se corren los tests en cada push y antes de cada merge, nunca como paso opcional.
5. Se evita testear detalles de implementacion interna; se testea comportamiento observable (input/output, efectos).

## Criterio junior

Un junior debe poder escribir un test unitario basico con `describe`/`it`/`expect` y ejecutar la suite completa con el comando del CLI.

## Criterio mid

Un mid debe estructurar suites de test legibles, usar `beforeEach` para evitar duplicacion de setup, y decidir que merece un test unitario y que merece un test de integracion.

## Criterio senior

Un senior debe definir la estrategia de testing del equipo (piramide de tests, umbral de cobertura, que se testea en CI vs local) y asegurar que los tests sean mantenibles, no solo que existan.

## Errores comunes

- Testear detalles de implementacion interna en vez de comportamiento observable, generando tests fragiles que se rompen con cualquier refactor.
- No correr tests en CI, dejando que bugs lleguen a produccion pese a tener tests escritos.
- Escribir tests con setup duplicado en cada `it` en vez de centralizarlo en `beforeEach`.
- Perseguir 100% de cobertura como meta en si misma, generando tests de bajo valor solo para subir el numero.

## Practica

- [ ] Escribe un test unitario simple para un servicio con `Test.createTestingModule()`.
- [ ] Configura `beforeEach` para evitar duplicacion de setup entre tests.
- [ ] Corre la suite completa de tests y revisa el reporte de cobertura.
- [ ] Identifica un test existente que testea implementacion en vez de comportamiento y refactorizalo.
- [ ] Anota una convencion propia de organizacion de tests para tu proyecto.

## Preguntas de repaso

- Por que los tests unitarios viven junto al codigo y los e2e en una carpeta separada?
- Que problema tiene testear detalles de implementacion interna?
- Para que sirve `Test.createTestingModule()` en vez de instanciar la clase directamente?
- Por que perseguir 100% de cobertura como meta puede ser contraproducente?
