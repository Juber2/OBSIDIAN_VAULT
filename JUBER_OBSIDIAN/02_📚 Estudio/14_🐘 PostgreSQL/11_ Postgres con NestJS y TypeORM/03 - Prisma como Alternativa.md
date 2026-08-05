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

# 03 - Prisma como Alternativa

## Que vas a aprender

En este capitulo vas a profundizar en Prisma (ya visto de forma introductoria en el libro de NestJS) especificamente con Postgres: como el schema de Prisma mapea a tipos especificos de Postgres, y sus capacidades y limitaciones frente a TypeORM para features avanzadas del motor.

## Explicacion

El archivo `schema.prisma` (ya visto en el libro de NestJS) soporta tipos especificos de Postgres directamente: `Json`/`Json?` mapea a `jsonb`, arrays nativos (`String[]`) mapean a arrays de Postgres (ver capitulo de Tipos de Datos Avanzados de este libro), y enums de Prisma mapean a ENUM nativos de Postgres. Prisma Migrate (el equivalente de las migrations de TypeORM) genera SQL automaticamente a partir de cambios en el schema, con el mismo principio de revision antes de aplicar ya visto en el tema anterior.

Una diferencia practica relevante: Prisma historicamente ha tenido soporte mas limitado para features muy especificas de Postgres (ciertos tipos de indice como GiST, algunas configuraciones avanzadas de constraints) comparado con TypeORM, aunque esto mejora con cada version. Para esos casos, Prisma permite migrations con SQL crudo de forma similar a TypeORM (editando el archivo de migration generado antes de aplicarlo), o usar `$queryRaw`/`$executeRaw` para consultas que aprovechan sintaxis especifica de Postgres no cubierta por el Query Builder de Prisma.

## Modelo mental

Pensa Prisma como un traductor mas moderno y con mejor autocompletado que TypeORM, pero que todavia esta poniendose al dia con algunos modismos muy especificos del idioma local (features avanzadas de Postgres). Para esos casos raros donde el traductor automatico no alcanza, siempre existe la opcion de escribir directamente en el idioma nativo (SQL crudo) para esa parte especifica.

## Ejemplo base

```prisma
model Producto {
  id       Int      @id @default(autoincrement())
  nombre   String
  tags     String[]
  metadata Json?
  estado   Estado   @default(ACTIVO)

  @@index([metadata], type: Gin)   // indice GIN explicito en el schema
}

enum Estado {
  ACTIVO
  INACTIVO
}
```

```typescript
// Query raw para aprovechar sintaxis especifica de Postgres no cubierta por Prisma Client
const resultado = await prisma.$queryRaw`
  SELECT * FROM productos WHERE metadata @> ${JSON.stringify({ en_oferta: true })}::jsonb
`;
```

## Como se usa en un proyecto real

1. Se declaran tipos especificos de Postgres (arrays, JSON, enums) directamente en el schema de Prisma, aprovechando su mapeo nativo.
2. Se usa `@@index([...], type: Gin)` para declarar indices GIN directamente en el schema cuando la version de Prisma lo soporta, evitando SQL crudo cuando no es necesario.
3. Se recurre a `$queryRaw`/`$executeRaw` para consultas que necesitan sintaxis muy especifica de Postgres (operadores de arrays, funciones de Full Text Search) no cubiertas por el Query Builder estandar.
4. Se revisa el SQL generado por `prisma migrate dev` antes de aplicarlo, con el mismo principio de revision ya visto en el tema anterior con TypeORM.
5. Se decide entre Prisma y TypeORM (ver capitulo de Base de Datos del libro de NestJS) considerando especificamente que tan avanzadas son las features de Postgres que el proyecto necesita aprovechar.

## Criterio junior

Un junior debe poder declarar tipos especificos de Postgres (array, Json, enum) en un schema de Prisma basico.

## Criterio mid

Un mid debe usar `$queryRaw` para consultas que necesitan sintaxis especifica de Postgres no cubierta por el Query Builder de Prisma.

## Criterio senior

Un senior debe decidir entre Prisma y TypeORM considerando especificamente el soporte de features avanzadas de Postgres necesarias para el proyecto, documentando esa decision.

## Errores comunes

- Asumir que Prisma soporta automaticamente cualquier feature avanzada de Postgres sin verificar la documentacion de la version especifica en uso.
- No revisar el SQL generado por `prisma migrate dev` antes de aplicarlo, arriesgando cambios no intencionados.
- Usar `$queryRaw` sin parametrizar correctamente los valores, arriesgando inyeccion SQL si se concatenan valores de usuario directamente.
- No considerar el soporte de features especificas de Postgres como criterio real al elegir entre Prisma y TypeORM para un proyecto nuevo.

## Practica

- [ ] Declara un modelo Prisma con tipos especificos de Postgres (array, Json, enum).
- [ ] Declara un indice GIN directamente en el schema si tu version de Prisma lo soporta.
- [ ] Escribe una consulta con `$queryRaw` que aproveche un operador especifico de Postgres (como `@>` sobre JSONB).
- [ ] Verifica que esa consulta esta correctamente parametrizada, sin riesgo de inyeccion SQL.
- [ ] Compara Prisma y TypeORM para un caso real de tu proyecto que necesite una feature avanzada de Postgres.

## Preguntas de repaso

- Que tipos especificos de Postgres soporta Prisma directamente en su schema?
- Cuando necesitarias usar `$queryRaw` en vez del Query Builder estandar de Prisma?
- Que riesgo de seguridad hay que evitar al usar `$queryRaw` con valores de usuario?
- Que criterio usarias para elegir entre Prisma y TypeORM segun las features de Postgres que tu proyecto necesita?
