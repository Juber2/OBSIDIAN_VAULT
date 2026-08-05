---
tags:
  - PostgreSQL
  - NestJS
  - Postgres-NestJS
Creado: 2026-07-29
Relacionado:
  - Postgres con NestJS y TypeORM
  - POSTGRESQL
---

# 05 - Testing con Postgres

## Que vas a aprender

En este capitulo vas a cerrar el capitulo consolidando estrategias de testing con Postgres real: bases de datos efimeras para tests de integracion (ya vistas en el libro de Docker), limpieza de estado entre tests, y cuando un fake en memoria (ya visto en el libro de NestJS) es suficiente versus cuando se necesita Postgres real.

## Explicacion

Para tests de integracion y e2e (ya vistos en profundidad en el libro de NestJS), usar **Postgres real** (tipicamente en un container Docker efimero, ver capitulo de Testing dentro de Containers del libro de Docker) da la maxima confianza porque valida el comportamiento real del motor: constraints, tipos de datos especificos como JSONB, comportamiento de transacciones. La estrategia de limpieza entre tests puede ser **truncar tablas** (`TRUNCATE ... RESTART IDENTITY CASCADE`, rapido) o envolver cada test en una **transaccion que se revierte** (`ROLLBACK`) al final, evitando necesitar truncar nada pero requiriendo cuidado con codigo que hace su propio manejo de transacciones.

Para tests **unitarios** de servicios (ya vistos en el libro de NestJS), un repositorio mockeado sigue siendo la eleccion correcta, sin necesitar Postgres real: la confianza que aporta Postgres real es especificamente sobre el comportamiento del motor y el mapeo de TypeORM, no sobre la logica de negocio en si misma, que ya se testea de forma aislada y rapida con mocks.

## Modelo mental

Pensa un test unitario con repositorio mockeado como probar la logica de un guion de teatro leyendolo en una mesa, sin actores ni escenario real. Un test de integracion con Postgres real es un ensayo general completo, con actores reales en el escenario real, verificando que todo funciona junto tal como sera la funcion de verdad. Ambos tipos de ensayo son necesarios, pero prueban cosas distintas.

## Ejemplo base

```typescript
// Test e2e con Postgres real en container Docker efimero (ver libro de Docker)
describe('ProductosController (e2e)', () => {
  let app: INestApplication;
  let dataSource: DataSource;

  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({ imports: [AppModule] }).compile();
    app = moduleFixture.createNestApplication();
    dataSource = moduleFixture.get(DataSource);
    await app.init();
  });

  afterEach(async () => {
    // Limpieza rapida entre tests
    await dataSource.query('TRUNCATE TABLE productos RESTART IDENTITY CASCADE');
  });

  afterAll(() => app.close());

  it('crea un producto con metadata JSONB', () => {
    return request(app.getHttpServer())
      .post('/productos')
      .send({ nombre: 'Camisa', metadata: { color: 'azul' } })
      .expect(201);
  });
});
```

```yaml
# docker-compose.test.yml: Postgres efimero solo para el pipeline de CI (ver libro de Docker)
services:
  postgres-test:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: test
```

## Como se usa en un proyecto real

1. Se usa Postgres real en container Docker efimero para tests de integracion y e2e, aplicando el mismo patron ya visto en el capitulo de Testing dentro de Containers del libro de Docker.
2. Se limpia el estado entre tests con `TRUNCATE ... RESTART IDENTITY CASCADE` (simple y rapido) o transacciones revertidas, segun el patron de acceso a datos del codigo bajo test.
3. Se mantienen los tests unitarios de servicios con repositorio mockeado (ver libro de NestJS), reservando Postgres real especificamente para verificar comportamiento del motor y mapeo de TypeORM.
4. Se testean especificamente casos que dependen de comportamiento real de Postgres (constraints, JSONB, Full Text Search) con Postgres real, dado que un mock no puede validar esas reglas.
5. Se integra este flujo en el pipeline de CI/CD, aplicando exactamente el patron ya visto en el capitulo de CI/CD del libro de Docker.

## Criterio junior

Un junior debe poder configurar un Postgres efimero para tests e2e siguiendo el patron ya visto en el libro de Docker.

## Criterio mid

Un mid debe implementar limpieza de estado entre tests eficientemente, y decidir cuando un test necesita Postgres real versus cuando un mock es suficiente.

## Criterio senior

Un senior debe diseñar la estrategia de testing de datos completa del equipo, balanceando la piramide de tests (unitarios con mock, integracion/e2e con Postgres real) para dar confianza sin sacrificar velocidad del pipeline.

## Errores comunes

- Usar Postgres real para todos los tests, incluso los que solo verifican logica de negocio pura, ralentizando innecesariamente la suite completa.
- No limpiar el estado entre tests, generando fallos intermitentes dependientes del orden de ejecucion.
- No testear especificamente comportamiento real de Postgres (constraints, JSONB) que un mock nunca podria validar correctamente.
- No integrar Postgres efimero en el pipeline de CI, dependiendo de una base de datos compartida que genera interferencia entre ejecuciones.

## Practica

- [ ] Configura un Postgres efimero en Docker para tests de integracion, aplicando el patron del libro de Docker.
- [ ] Implementa limpieza de estado entre tests con `TRUNCATE` o transacciones revertidas.
- [ ] Escribe un test que verifique especificamente un constraint o comportamiento de JSONB que solo Postgres real puede validar.
- [ ] Compara ese test con el equivalente unitario usando un repositorio mockeado.
- [ ] Integra esta estrategia de testing en un pipeline de CI de ejemplo.

## Preguntas de repaso

- Que ventaja especifica da testear con Postgres real sobre un repositorio mockeado?
- Que estrategias existen para limpiar el estado de la base de datos entre tests?
- Cuando un test unitario con mock sigue siendo la eleccion correcta en vez de usar Postgres real?
- Como integrarias Postgres efimero en un pipeline de CI, conectando con lo ya visto en el libro de Docker?

## Cierre del libro

Completar este capitulo cierra el recorrido de PostgreSQL desde fundamentos hasta produccion: dominio de SQL, modelado relacional solido, indices y optimizacion basados en evidencia, transacciones y concurrencia con MVCC, programabilidad con funciones/triggers, tipos avanzados que aprovechan lo mejor de Postgres, administracion y backup confiables, alta disponibilidad, y la integracion completa con NestJS/TypeORM que conecta todo este conocimiento con el resto del ecosistema de este vault.
