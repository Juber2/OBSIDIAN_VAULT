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

# 02 - Migrations con TypeORM

## Que vas a aprender

En este capitulo vas a profundizar en migrations de TypeORM (ya vistas en el libro de NestJS) desde el lado de Postgres: como TypeORM traduce decoradores de entidad a SQL real de Postgres, y como aprovechar features especificas del motor (tipos JSONB, ENUM, indices GIN) dentro de una migration.

## Explicacion

Cuando TypeORM genera una migration automaticamente (`migration:generate`), traduce los decoradores de la entidad a instrucciones SQL especificas de Postgres: un `@Column({ type: 'jsonb' })` genera una columna `jsonb` real (ver capitulo de Tipos de Datos Avanzados), un `@Column({ type: 'enum', enum: EstadoPedido })` genera un `CREATE TYPE ... AS ENUM` (ver capitulo de Tipos de Datos Avanzados), y un `@Index()` con opciones especificas puede generar un indice GIN o parcial (ver capitulo de Indices) si se configura correctamente.

Es comun necesitar **editar manualmente** una migration autogenerada para aprovechar features de Postgres que TypeORM no expone completamente via decoradores: agregar un indice GIN explicito, un `CHECK` constraint complejo, o un trigger (ver capitulo de Funciones y Triggers), escribiendo SQL crudo dentro del metodo `up()`/`down()` de la migration cuando el decorador no cubre el caso.

## Modelo mental

Pensa el generador automatico de migrations de TypeORM como un traductor que convierte bien el 90% de un idioma (los decoradores comunes), pero que a veces necesita que un hablante nativo (vos, escribiendo SQL crudo dentro de la migration) revise y corrija manualmente las expresiones idiomaticas especificas de Postgres que el traductor automatico no captura perfectamente.

## Ejemplo base

```typescript
@Entity()
export class Producto {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ type: 'jsonb', nullable: true })
  metadata: Record<string, any>;

  @Column({ type: 'enum', enum: ['activo', 'inactivo'] })
  estado: string;
}
```

```typescript
// Migration con SQL crudo para features que el decorador no cubre completamente
export class AddGinIndexToMetadata1234567890 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(
      `CREATE INDEX idx_productos_metadata ON productos USING gin (metadata)`,
    );
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_productos_metadata`);
  }
}
```

## Como se usa en un proyecto real

1. Se revisa siempre el SQL generado automaticamente por `migration:generate` antes de aplicarlo, verificando que refleja correctamente la intencion (aplicando el mismo principio de "revisar antes de aplicar" ya visto en varios libros de esta serie).
2. Se agrega SQL crudo dentro de una migration cuando se necesita una feature especifica de Postgres (indice GIN, constraint complejo) que los decoradores estandar de TypeORM no exponen directamente.
3. Se prueba localmente cada migration en ambas direcciones (`up` y `down`) antes de mergear, aplicando el mismo principio ya visto en el libro de NestJS.
4. Se documenta en la propia migration (con un comentario breve) por que se necesito SQL crudo, ayudando a futuros mantenedores a entender la razon.
5. Se conecta esto directamente con el capitulo de Migrations del libro de NestJS, ahora con el conocimiento profundo de Postgres de este libro para aprovechar mejor sus features especificas.

## Criterio junior

Un junior debe poder generar y revisar una migration automatica de TypeORM antes de aplicarla.

## Criterio mid

Un mid debe agregar SQL crudo dentro de una migration para features de Postgres no cubiertas completamente por decoradores (indices GIN, constraints complejos).

## Criterio senior

Un senior debe revisar en code review que las migrations aprovechan correctamente features de Postgres relevantes para performance y correccion, documentando decisiones no obvias de SQL crudo.

## Errores comunes

- Confiar ciegamente en el SQL autogenerado sin revisarlo, aplicando cambios que no reflejan exactamente la intencion real.
- No aprovechar indices GIN o parciales (ver capitulo de Indices) cuando el caso lo justifica, porque el decorador estandar de TypeORM no los genera automaticamente sin configuracion adicional.
- No documentar por que una migration necesito SQL crudo, dificultando el entendimiento futuro de esa decision.
- No probar el `down` de migrations con SQL crudo, arriesgando que el rollback falle justo cuando mas se necesita.

## Practica

- [ ] Genera una migration automatica a partir de una entidad con una columna `jsonb` y revisa el SQL generado.
- [ ] Agrega manualmente un indice GIN a esa migration usando SQL crudo.
- [ ] Prueba el `up` y `down` de esa migration completa.
- [ ] Identifica una entidad de tu proyecto NestJS real que se beneficiaria de una feature especifica de Postgres no cubierta por decoradores.
- [ ] Documenta esa decision de SQL crudo con un comentario breve en la migration.

## Preguntas de repaso

- Que traduce TypeORM automaticamente al generar una migration a partir de decoradores de entidad?
- Cuando necesitarias agregar SQL crudo dentro de una migration de TypeORM?
- Por que es importante revisar el SQL autogenerado antes de aplicarlo?
- Que documentarias en una migration que usa SQL crudo para ayudar a futuros mantenedores?
