---
tags:
  - Java
  - JDBC
Creado: 2026-05-21
Relacionado:
  - Bases de Datos Java
---

# JDBC y Connection Pools

JDBC es la API base de Java para hablar con bases de datos relacionales. Spring Data y JPA se apoyan indirectamente en esta capa.

## JDBC basico

```java
try (Connection connection = dataSource.getConnection();
     PreparedStatement statement = connection.prepareStatement(
         "select id, email from users where id = ?")) {

    statement.setLong(1, id);
    try (ResultSet rs = statement.executeQuery()) {
        if (rs.next()) {
            return new User(rs.getLong("id"), rs.getString("email"));
        }
    }
}
```

## Connection pool

Abrir conexiones es caro. Un pool mantiene conexiones reutilizables. En Spring Boot, HikariCP es el default.

## Configuracion

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
```

## Checklist

- [ ] Uso `PreparedStatement`.
- [ ] Cierro recursos con try-with-resources.
- [ ] Dimensiono pool segun DB y trafico.
- [ ] No abro conexiones manuales en cada metodo si Spring ya gestiona transacciones.
