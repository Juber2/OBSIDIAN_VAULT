---
tags:
  - Java
  - SQL
  - Performance
Creado: 2026-05-21
Relacionado:
  - Bases de Datos Java
---

# Indices y Queries Lentas

Muchas mejoras de performance en Java se resuelven en la base de datos.

## Indice

Un indice acelera busquedas, filtros y ordenamientos, pero aumenta costo de escritura y ocupa espacio.

```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

## EXPLAIN

Usa `EXPLAIN` o `EXPLAIN ANALYZE` para ver como la base ejecuta una query.

## Senales de query lenta

- Full table scan en tablas grandes.
- Filtros sobre columnas sin indice.
- `LIKE '%texto'`.
- Paginacion con offsets enormes.
- N+1 desde JPA.

## Checklist

- [ ] Reviso plan de ejecucion.
- [ ] Creo indices para patrones reales de consulta.
- [ ] No indexo todo.
- [ ] Mido antes y despues.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

