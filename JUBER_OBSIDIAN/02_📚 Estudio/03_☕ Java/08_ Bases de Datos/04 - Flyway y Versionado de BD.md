---
tags:
  - Java
  - Flyway
Creado: 2026-05-21
Relacionado:
  - Bases de Datos Java
---

# Flyway y Versionado de BD

Flyway versiona cambios de base de datos con scripts SQL.

## Convencion

```text
src/main/resources/db/migration/
  V1__create_users_table.sql
  V2__add_status_to_users.sql
```

## Ejemplo

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(120) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

## Buenas practicas

- No edites migraciones ya aplicadas en ambientes compartidos.
- Crea una migracion nueva.
- Manten scripts revisables.
- Evita cambios destructivos sin plan.

## Checklist

- [ ] Cada cambio de esquema esta versionado.
- [ ] La app arranca contra una BD vacia.
- [ ] No uso `ddl-auto=update` en produccion.
- [ ] Los scripts son idempotentes cuando aplica.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

