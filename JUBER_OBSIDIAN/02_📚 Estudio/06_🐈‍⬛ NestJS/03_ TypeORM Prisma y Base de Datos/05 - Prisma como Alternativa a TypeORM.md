---
tags:
  - NestJS
  - TypeScript
  - Nest-Datos
Creado: 2026-07-27
Relacionado:
  - TypeORM Prisma y Base de Datos
  - NESTJS
---

# 05 - Prisma como Alternativa a TypeORM

## Que vas a aprender

En este capitulo vas a entender **Prisma** como alternativa a TypeORM: su enfoque schema-first, el cliente tipado autogenerado, y como integrarlo dentro de un modulo NestJS de forma limpia.

## Explicacion

A diferencia de TypeORM (entidades como clases decoradas), Prisma define el esquema en un archivo `schema.prisma` propio, y a partir de ahi genera un cliente TypeScript completamente tipado (`PrismaClient`) con autocompletado real basado en el esquema. Esto elimina una clase de errores (typos en nombres de columnas, tipos incorrectos) que con TypeORM solo se detectan en runtime.

La forma recomendada de integrar Prisma en Nest es crear un `PrismaService` que extiende `PrismaClient` e implementa `OnModuleInit`/`OnModuleDestroy` para manejar la conexion correctamente dentro del ciclo de vida de Nest.

## Modelo mental

Pensa Prisma como un compilador de tu esquema de datos: escribis el modelo una vez en `schema.prisma`, y Prisma genera automaticamente un cliente que "sabe" exactamente que tablas, columnas y relaciones existen, con tipos TypeScript exactos, sin que vos escribas esas clases a mano.

## Ejemplo base

```prisma
// schema.prisma
model User {
  id    String @id @default(uuid())
  email String @unique
  orders Order[]
}
```

```typescript
@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await this.$connect();
  }
  async onModuleDestroy() {
    await this.$disconnect();
  }
}

@Injectable()
export class UsersService {
  constructor(private readonly prisma: PrismaService) {}

  findByEmail(email: string) {
    return this.prisma.user.findUnique({ where: { email } });
  }
}
```

## Como se usa en un proyecto real

1. Se define el esquema completo en `schema.prisma` como fuente de verdad.
2. Se genera el cliente (`npx prisma generate`) como parte del proceso de build.
3. Se crea un `PrismaService` unico, inyectado donde se necesite acceso a datos.
4. Las migraciones se manejan con `prisma migrate dev`/`prisma migrate deploy`, analogas a las de TypeORM pero generadas a partir del schema.
5. Se decide como equipo entre TypeORM y Prisma al inicio del proyecto; mezclar ambos en el mismo dominio de datos genera confusion e inconsistencia.

## Criterio junior

Un junior debe poder definir un modelo simple en `schema.prisma`, generar el cliente, y hacer consultas basicas con `PrismaService`.

## Criterio mid

Un mid debe manejar migraciones con Prisma Migrate, modelar relaciones complejas en el schema, y decidir cuando usar transacciones interactivas de Prisma (`$transaction`).

## Criterio senior

Un senior debe justificar la eleccion entre TypeORM y Prisma para un proyecto nuevo segun el contexto del equipo (preferencia por tipado estricto autogenerado vs flexibilidad de entidades como clases), y definir la estrategia de migraciones para equipos grandes.

## Errores comunes

- Mezclar TypeORM y Prisma en el mismo proyecto sin una razon clara, duplicando la fuente de verdad del esquema.
- No regenerar el cliente Prisma despues de cambiar el schema y arrastrar tipos desactualizados.
- Usar el `PrismaService` sin manejar correctamente `onModuleInit`/`onModuleDestroy`, dejando conexiones abiertas.
- Ignorar transacciones interactivas para operaciones que modifican varias tablas relacionadas.

## Practica

- [ ] Define un schema Prisma con dos modelos relacionados.
- [ ] Crea el `PrismaService` con el ciclo de vida correcto.
- [ ] Escribe una consulta con relacion incluida (`include`).
- [ ] Compara mentalmente la experiencia de tipado entre TypeORM y Prisma.
- [ ] Anota una decision propia: en que proyecto elegirias Prisma y en cual TypeORM.

## Preguntas de repaso

- Que diferencia fundamental hay entre el enfoque de TypeORM y el de Prisma?
- Por que se recomienda encapsular `PrismaClient` en un `PrismaService`?
- Que ventaja de tipado ofrece el cliente generado de Prisma sobre entidades escritas a mano?
- Que riesgo hay en mezclar dos ORMs en el mismo dominio de datos?
