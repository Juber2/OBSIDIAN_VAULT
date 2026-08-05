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

# 04 - Migrations y Seeders

## Que vas a aprender

En este capitulo vas a aprender a manejar cambios de esquema de base de datos de forma controlada con **migrations** de TypeORM, y a poblar datos de prueba con **seeders**, sin depender nunca de `synchronize: true`.

## Explicacion

Una migration es un archivo versionado que describe un cambio de esquema (crear tabla, agregar columna, crear indice) con su version de "ida" (`up`) y "vuelta" (`down`). Son el mecanismo seguro y auditable para evolucionar la base de datos en cualquier ambiente compartido, a diferencia de `synchronize: true` que aplica cambios automaticos sin historial ni control.

Los seeders son scripts que insertan datos de prueba o datos base necesarios (roles por defecto, usuario admin inicial) de forma repetible, separados completamente de las migrations de esquema.

## Modelo mental

Pensa las migrations como un historial de commits, pero para la estructura de la base de datos: cada migration es un paso reproducible que cualquier desarrollador o el pipeline de CI puede aplicar, en orden, para llegar exactamente al mismo esquema.

## Ejemplo base

```typescript
export class AddPhoneToUsers1720000000000 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`ALTER TABLE "users" ADD "phone" varchar`);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`ALTER TABLE "users" DROP COLUMN "phone"`);
  }
}
```

```bash
npx typeorm migration:generate src/migrations/AddPhoneToUsers -d src/data-source.ts
npx typeorm migration:run -d src/data-source.ts
```

## Como se usa en un proyecto real

1. Toda modificacion de esquema pasa por una migration versionada y commiteada al repositorio.
2. Las migrations se aplican automaticamente en el pipeline de CI/CD antes de desplegar la nueva version de la app.
3. Cada migration se prueba localmente en ambas direcciones (`up` y `down`) antes de mergear.
4. Los seeders se separan claramente en "datos base necesarios para que la app funcione" vs "datos de prueba solo para desarrollo".
5. Se evita mezclar cambios de esquema grandes con datos: una migration hace una cosa, con un nombre descriptivo.

## Criterio junior

Un junior debe poder generar y correr una migration simple, y entender por que `synchronize: true` no es una alternativa valida a las migrations en un ambiente compartido.

## Criterio mid

Un mid debe escribir migrations manuales para cambios que el generador automatico no captura bien (renombrar columnas preservando datos, migraciones de datos), y mantener seeders organizados por ambiente.

## Criterio senior

Un senior debe definir la estrategia de migrations del equipo (revision obligatoria en PR, migrations reversibles, ventanas de mantenimiento para cambios riesgosos en tablas grandes) y anticipar el impacto de una migration en produccion bajo carga (locks, tiempo de ejecucion).

## Errores comunes

- Usar `synchronize: true` en un ambiente compartido y perder control sobre el historial de cambios de esquema.
- Escribir una migration que no tiene `down` funcional, imposibilitando el rollback.
- Mezclar seeders de datos de prueba con datos base necesarios para produccion.
- Ejecutar una migration que bloquea una tabla grande en produccion sin planificar una ventana de mantenimiento.

## Practica

- [ ] Genera una migration para agregar una columna nueva a una entidad existente.
- [ ] Verifica que el `down` de esa migration revierta correctamente el cambio.
- [ ] Crea un seeder que inserte un usuario admin por defecto.
- [ ] Simula correr las migrations en un pipeline de CI local.
- [ ] Anota una checklist de revision de migrations para tu equipo.

## Preguntas de repaso

- Por que `synchronize: true` es peligroso en un ambiente compartido?
- Que deberia contener el metodo `down` de una migration?
- Como separarias seeders de datos base de seeders de datos de prueba?
- Que riesgo tiene aplicar una migration pesada sobre una tabla grande en produccion?
