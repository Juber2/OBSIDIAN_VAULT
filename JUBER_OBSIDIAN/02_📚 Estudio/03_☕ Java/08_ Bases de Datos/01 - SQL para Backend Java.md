---
tags:
  - Java
  - SQL
Creado: 2026-05-21
Relacionado:
  - Bases de Datos Java
---

# SQL para Backend Java

Un backend Java senior debe entender SQL aunque use JPA.

## Operaciones clave

```sql
SELECT id, name, email
FROM users
WHERE status = 'ACTIVE'
ORDER BY created_at DESC
LIMIT 20 OFFSET 0;
```

## Joins

```sql
SELECT o.id, o.created_at, u.email
FROM orders o
JOIN users u ON u.id = o.user_id
WHERE o.status = 'PAID';
```

## Conceptos indispensables

- Normalizacion.
- Claves primarias y foraneas.
- Indices.
- Transacciones.
- Bloqueos.
- Paginacion.

## Checklist

- [ ] Escribo joins sin depender del ORM.
- [ ] Entiendo diferencia entre `WHERE` y `HAVING`.
- [ ] Uso paginacion.
- [ ] Evito traer columnas innecesarias.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

