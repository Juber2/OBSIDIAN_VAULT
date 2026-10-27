---
tags:
  - Java
  - Spring
  - JPA
Creado: 2026-05-21
Relacionado:
  - Spring Data
Palabras clave:
  - Flyway, Liquibase, migraciones, versionado de BD, schema evolution
---

## 🧠 Por qué usar migraciones

- `ddl-auto=update` en producción es peligroso — puede perder datos
- Las migraciones versionan los cambios de BD como el código
- Permiten reproducir el estado exacto de la BD en cualquier entorno

## Flyway (más simple — recomendado para empezar)

### Dependencia

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<!-- Para PostgreSQL -->
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

### Configuración

```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
spring.flyway.baseline-on-migrate=true  # para BD existentes
spring.jpa.hibernate.ddl-auto=validate  # Flyway maneja el schema, Hibernate solo valida
```

### Estructura de archivos

```
src/main/resources/
└── db/
    └── migration/
        ├── V1__crear_tabla_usuarios.sql
        ├── V2__agregar_columna_telefono.sql
        ├── V3__crear_tabla_pedidos.sql
        └── V4__agregar_indices.sql
```

### Convención de nombres

```
V{versión}__{descripción}.sql

V1__crear_tabla_usuarios.sql      → versión 1
V2__agregar_campo_telefono.sql    → versión 2
V1.1__fix_constraint.sql          → versión 1.1
R__datos_semilla.sql              → Repeatable (se ejecuta cuando cambia)
U3__deshacer_pedidos.sql          → Undo (requiere Flyway Teams)
```

### Scripts SQL de ejemplo

```sql
-- V1__crear_tabla_usuarios.sql
CREATE TABLE usuarios (
    id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT true,
    creado_en TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP
);

CREATE INDEX idx_usuarios_email ON usuarios(email);

-- V2__crear_tabla_pedidos.sql
CREATE TABLE pedidos (
    id BIGSERIAL PRIMARY KEY,
    usuario_id BIGINT NOT NULL REFERENCES usuarios(id),
    total DECIMAL(10,2) NOT NULL,
    estado VARCHAR(50) NOT NULL DEFAULT 'PENDIENTE',
    creado_en TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT chk_total_positivo CHECK (total > 0)
);

-- V3__agregar_columna_telefono.sql
ALTER TABLE usuarios ADD COLUMN telefono VARCHAR(20);

-- V4__datos_iniciales.sql
INSERT INTO usuarios (nombre, email, password_hash)
VALUES ('Admin', 'admin@app.com', '$2a$10$...');
```

## Liquibase (más potente — XML/YAML/JSON)

### Dependencia

```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

### Configuración

```properties
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.xml
```

### Changelog en YAML

```yaml
# db/changelog/db.changelog-master.yaml
databaseChangeLog:
  - include:
      file: db/changelog/001-crear-usuarios.yaml
  - include:
      file: db/changelog/002-crear-pedidos.yaml
```

```yaml
# 001-crear-usuarios.yaml
databaseChangeLog:
  - changeSet:
      id: 001
      author: juber
      changes:
        - createTable:
            tableName: usuarios
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: nombre
                  type: VARCHAR(100)
                  constraints:
                    nullable: false
```

## Flyway vs Liquibase

| | Flyway | Liquibase |
|--|--------|-----------|
| Formato | SQL puro | SQL, XML, YAML, JSON |
| Simplicidad | ✅ Más simple | Más complejo |
| Rollback | No (requiere Pro) | ✅ Soporte nativo |
| Difftool | No | ✅ diffChangelog |
| Uso típico | Proyectos simples-medianos | Proyectos grandes/enterprise |

## 📌 Notas Rápidas
- Una migración de Flyway aplicada **nunca se edita** — si tiene un error, se crea una nueva migración que lo corrige; editar un archivo ya aplicado rompe el checksum y falla en otros ambientes.
- `baseline-on-migrate=true` es necesario solo la primera vez que se introduce Flyway en una BD que ya tenía tablas creadas manualmente — evita que Flyway intente recrear lo que ya existe.
- Con `ddl-auto=validate` + Flyway, Hibernate deja de ser la fuente de verdad del schema — las migraciones SQL lo son, Hibernate solo confirma que las entidades coinciden.
- Este mismo mecanismo (Flyway) es el que ya usás en Quarkus con Panache (ver [[05 - Panache y Persistencia]]) — es independiente del framework, funciona igual en ambos ecosistemas.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

