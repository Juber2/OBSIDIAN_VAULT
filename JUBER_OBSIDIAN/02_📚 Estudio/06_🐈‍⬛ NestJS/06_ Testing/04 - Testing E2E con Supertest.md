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

# 04 - Testing E2E con Supertest

## Que vas a aprender

En este capitulo vas a aprender a escribir tests **end-to-end** en NestJS, levantando la aplicacion completa en memoria y haciendo requests HTTP reales con **Supertest** contra una base de datos de test.

## Explicacion

Un test e2e verifica el sistema completo tal como lo usaria un cliente real: levanta la aplicacion Nest (`app.init()`), y hace requests HTTP de verdad contra sus endpoints con Supertest, verificando status code, body y efectos secundarios (que el dato realmente se guardo en la base). A diferencia de los tests unitarios, aca no se mockean las dependencias internas del sistema; si se mockean servicios externos verdaderamente externos (pasarelas de pago, envio de emails).

Estos tests son mas lentos que los unitarios pero dan mayor confianza porque prueban la integracion real entre controller, pipes, guards, servicio y base de datos. Se corren contra una base de datos de test separada, que se limpia entre tests o entre suites.

## Modelo mental

Pensa el test e2e como un usuario robot que realmente toca la aplicacion desde afuera: hace un POST real, recibe una respuesta real, y podemos verificar que efectivamente quedo un registro en la base de datos, igual que pasaria en produccion.

## Ejemplo base

```typescript
describe('UsersController (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();
    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterAll(() => app.close());

  it('/users (POST) crea un usuario', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({ email: 'test@test.com', password: 'password123' })
      .expect(201)
      .expect((res) => {
        expect(res.body.email).toBe('test@test.com');
        expect(res.body.password).toBeUndefined();
      });
  });
});
```

## Como se usa en un proyecto real

1. Se usa una base de datos de test dedicada (contenedor Docker efimero o base separada), nunca la de desarrollo o produccion.
2. Se limpia el estado de la base entre tests o entre suites para evitar que un test dependa del orden de ejecucion de otro.
3. Se testean los flujos criticos de negocio de punta a punta (registro, login, checkout), no cada combinacion posible de cada endpoint.
4. Se mockean solo las dependencias verdaderamente externas (proveedores de pago, servicios de email), no las internas del sistema.
5. Se corren en un pipeline de CI separado de los unitarios, dado que son mas lentos.

## Criterio junior

Un junior debe poder escribir un test e2e basico que levante la app y verifique un endpoint simple con Supertest.

## Criterio mid

Un mid debe configurar una base de datos de test aislada, limpiar el estado entre tests, y testear flujos completos de varios pasos (crear, luego consultar, luego actualizar).

## Criterio senior

Un senior debe definir la estrategia de testing e2e del equipo (que flujos se cubren, como se aislan de produccion, como se integran en CI/CD sin ralentizar demasiado el pipeline) y balancear la piramide de tests para no depender excesivamente de e2e.

## Errores comunes

- Correr tests e2e contra la base de datos de desarrollo, contaminando datos reales.
- No limpiar el estado entre tests, generando fallos intermitentes segun el orden de ejecucion.
- Intentar cubrir cada combinacion posible con e2e en vez de unitarios, volviendo la suite lentisima.
- No cerrar la aplicacion (`app.close()`) al final de la suite, dejando conexiones abiertas que cuelgan el proceso de test.

## Practica

- [ ] Configura una base de datos de test separada para tus tests e2e.
- [ ] Escribe un test e2e que cree un recurso y luego lo consulte.
- [ ] Implementa limpieza de estado entre tests (transaccion revertida o truncate de tablas).
- [ ] Verifica que un endpoint protegido rechace correctamente una request sin autenticacion.
- [ ] Anota que flujos de tu dominio merecen cobertura e2e prioritaria.

## Preguntas de repaso

- Que diferencia de confianza da un test e2e frente a un test unitario?
- Por que un test e2e nunca deberia correr contra la base de datos de produccion?
- Como evitarias que el orden de ejecucion de los tests afecte sus resultados?
- Que dependencias externas si conviene mockear incluso en un test e2e?
