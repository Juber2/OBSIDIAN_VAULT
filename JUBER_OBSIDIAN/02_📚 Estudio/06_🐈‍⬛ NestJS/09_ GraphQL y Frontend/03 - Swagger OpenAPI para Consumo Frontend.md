---
tags:
  - NestJS
  - TypeScript
  - Nest-GraphQL
Creado: 2026-07-27
Relacionado:
  - GraphQL y Frontend
  - NESTJS
---

# 03 - Swagger OpenAPI para Consumo Frontend

## Que vas a aprender

En este capitulo vas a aprender a aprovechar la especificacion **OpenAPI** generada por Swagger (ver capitulo de APIs REST) para generar automaticamente **clientes tipados** consumibles desde un frontend Angular/React, cerrando el circulo entre backend y frontend sin escribir tipos a mano dos veces.

## Explicacion

Una vez que el backend expone una especificacion OpenAPI completa y precisa (`@nestjs/swagger`), esa especificacion se puede usar como entrada para generadores de clientes (`openapi-generator`, `orval`, `openapi-typescript`) que producen automaticamente interfaces TypeScript y funciones de cliente HTTP tipadas para el frontend. Esto elimina la duplicacion manual de DTOs entre backend y frontend, y hace que un cambio de contrato en el backend se refleje como un error de compilacion en el frontend si no se actualiza el cliente.

Esto es especialmente valioso en equipos donde el mismo desarrollador (o equipo) mantiene tanto NestJS como Angular, permitiendo un flujo de "cambio el DTO en el backend, regenero el cliente, TypeScript en el frontend me marca donde tengo que actualizar".

## Modelo mental

Pensa la especificacion OpenAPI como un plano tecnico compartido: en vez de que el frontend "adivine" o copie a mano la forma de los datos que el backend expone, un generador lee el plano oficial y construye automaticamente el codigo de consumo, siempre sincronizado con la realidad del backend.

## Ejemplo base

```bash
# Backend expone /docs-json con la spec OpenAPI (ademas de /docs con la UI)

# Frontend genera un cliente tipado a partir de esa spec
npx openapi-typescript http://localhost:3000/docs-json -o src/api/schema.d.ts
```

```typescript
// Frontend: uso tipado automaticamente, sin escribir el DTO a mano
import type { paths } from './api/schema';
type CreateUserBody = paths['/users']['post']['requestBody']['content']['application/json'];
```

## Como se usa en un proyecto real

1. El backend expone un endpoint dedicado con la especificacion OpenAPI en JSON (ademas de la UI de Swagger para humanos).
2. El pipeline de frontend regenera el cliente tipado como parte del build o de un script manual antes de cada release.
3. Se versiona el archivo generado (o se regenera en CI) para detectar cambios de contrato no intencionales.
4. Se coordina con el equipo frontend el momento de romper un contrato (versionado de API, ver capitulo de APIs REST) para no generar builds rotos sorpresivos.
5. Se documentan bien los DTOs en el backend (`@ApiProperty` con descripciones) porque esa documentacion termina siendo la que ve el equipo frontend en el cliente generado.

## Criterio junior

Un junior debe entender que la especificacion OpenAPI puede usarse para generar codigo de consumo automatico, y saber ejecutar un generador basico contra un backend Nest.

## Criterio mid

Un mid debe integrar la generacion de clientes en el flujo de trabajo del equipo (script manual o paso de CI), y coordinar cambios de contrato con el equipo frontend.

## Criterio senior

Un senior debe decidir la estrategia de sincronizacion de contrato entre backend y frontend para toda la organizacion (generacion automatica en CI, monorepo con tipos compartidos, o documentacion manual) segun el tamaño y estructura de los equipos.

## Errores comunes

- Mantener DTOs escritos a mano por separado en frontend y backend, permitiendo que se desincronicen silenciosamente con el tiempo.
- No exponer la especificacion OpenAPI en JSON, forzando a copiar tipos manualmente desde la UI de Swagger.
- Romper un contrato de API sin coordinar con el frontend, generando errores en produccion del lado cliente.
- No regenerar el cliente frontend despues de un cambio de contrato, quedando con tipos desactualizados silenciosamente.

## Practica

- [ ] Expone la especificacion OpenAPI en JSON desde un proyecto Nest de practica.
- [ ] Genera un cliente TypeScript tipado a partir de esa especificacion.
- [ ] Simula un cambio de contrato en el backend y observa el error de tipos en el cliente generado.
- [ ] Automatiza la regeneracion del cliente como parte de un script de build.
- [ ] Anota como coordinarias cambios de contrato entre backend y frontend en tu equipo.

## Preguntas de repaso

- Que ventaja da generar el cliente frontend a partir de la especificacion OpenAPI en vez de escribir DTOs a mano?
- Como te enterarias, del lado frontend, de que el backend rompio un contrato?
- Que informacion del backend deberia documentarse bien para que el cliente generado sea util?
- En que momento del flujo de trabajo conviene regenerar el cliente tipado?
