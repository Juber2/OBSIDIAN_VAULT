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

# 05 - Versionado y Documentacion con Swagger

## Que vas a aprender

En este capitulo vas a aprender a **versionar** una API NestJS de forma controlada, y a generar documentacion interactiva y siempre actualizada con `@nestjs/swagger` (OpenAPI).

## Explicacion

El versionado de API permite evolucionar contratos sin romper clientes existentes. Nest soporta versionado por URI (`/v1/users`, `/v2/users`), por header, o por media type, configurable de forma global con `app.enableVersioning()`. La regla practica es: un cambio incompatible (romper un campo, cambiar un tipo, quitar un endpoint) exige una nueva version; un cambio aditivo (agregar un campo opcional) no.

`@nestjs/swagger` genera documentacion OpenAPI directamente a partir de los decoradores ya presentes en controllers y DTOs (`@ApiTags`, `@ApiOperation`, `@ApiProperty`), lo que mantiene la documentacion sincronizada con el codigo real en vez de un documento separado que se desactualiza.

## Modelo mental

Pensa el versionado como contratos con fecha de vigencia: los clientes que ya integraron `/v1` siguen funcionando exactamente igual aunque el equipo cambie cosas en `/v2`. La documentacion Swagger es el manual de ese contrato, generado automaticamente para que nunca quede desactualizado respecto al codigo.

## Ejemplo base

```typescript
// main.ts
app.enableVersioning({ type: VersioningType.URI, defaultVersion: '1' });

const config = new DocumentBuilder()
  .setTitle('API')
  .setVersion('1.0')
  .addBearerAuth()
  .build();
const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('docs', app, document);

@Controller({ path: 'users', version: '1' })
@ApiTags('users')
export class UsersController {
  @ApiOperation({ summary: 'Lista usuarios' })
  @Get()
  findAll() {}
}
```

## Como se usa en un proyecto real

1. Se activa versionado desde el inicio del proyecto, aunque solo exista `v1`, para no tener que migrar despues bajo presion.
2. Se documenta cada endpoint publico con decoradores de Swagger a medida que se escribe, no como tarea separada al final.
3. Swagger UI se protege o se deshabilita en produccion si expone informacion sensible de la API interna.
4. Se define una politica de deprecacion: cuanto tiempo convive una version vieja antes de eliminarse.
5. Se documentan tambien los codigos de error posibles de cada endpoint (`@ApiResponse`) para que el consumidor sepa que esperar.

## Criterio junior

Un junior debe poder documentar un endpoint basico con decoradores de Swagger y entender para que sirve el versionado por URI.

## Criterio mid

Un mid debe mantener la documentacion Swagger sincronizada con cambios de contrato, y aplicar versionado cuando un cambio es incompatible con clientes existentes.

## Criterio senior

Un senior debe definir la politica de versionado y deprecacion de la API para todo el equipo, y decidir cuando conviene versionar por URI, header o negociacion de contenido segun los consumidores de la API.

## Errores comunes

- Cambiar el contrato de un endpoint existente sin versionar, rompiendo integraciones de clientes ya en produccion.
- Dejar la documentacion Swagger desactualizada porque se escribe "despues" en vez de junto con el codigo.
- Exponer Swagger UI publicamente en produccion sin evaluar si eso filtra informacion sensible de la arquitectura interna.
- No definir una politica clara de cuanto tiempo se mantiene una version vieja antes de eliminarla.

## Practica

- [ ] Activa versionado por URI en un proyecto Nest.
- [ ] Documenta un controller completo con decoradores de Swagger, incluyendo respuestas de error.
- [ ] Simula un cambio incompatible de contrato y crea una nueva version del endpoint.
- [ ] Revisa si Swagger UI deberia estar protegido en produccion para tu proyecto.
- [ ] Anota una politica propia de deprecacion de versiones de API.

## Preguntas de repaso

- Que diferencia hay entre un cambio aditivo y uno incompatible en un contrato de API?
- Por que versionar desde el inicio, aunque solo exista una version, facilita el futuro?
- Que riesgo tiene exponer Swagger UI sin proteccion en produccion?
- Como mantendrias la documentacion Swagger sincronizada con el codigo a lo largo del tiempo?
