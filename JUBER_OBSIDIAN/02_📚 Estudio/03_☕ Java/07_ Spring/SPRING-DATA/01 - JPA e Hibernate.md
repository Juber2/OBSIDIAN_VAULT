---
tags:
  - Java
  - Spring
  - JPA
Creado: 2026-05-21
Relacionado:
  - Spring Data
Palabras clave:
  - JPA, Hibernate, ORM, EntityManager, persistencia, mapeo objeto-relacional
---

## 🧠 Concepto

**JPA (Jakarta Persistence API)** es la especificación estándar de Java para ORM. **Hibernate** es la implementación más popular de JPA.

```
Tu código Java ← JPA (contrato) → Hibernate (implementación) → Base de Datos
```

## Dependencia

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

## Configuración

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/midb
spring.datasource.username=postgres
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.open-in-view=false  # SIEMPRE false en producción
```

## Estados de una entidad JPA

```
Transient → Managed → Detached → Removed

Transient:  new Usuario() — no conocido por JPA, no en BD
Managed:    entityManager.persist(u) — JPA lo rastrea, sincroniza con BD
Detached:   entityManager.detach(u) — ya no rastreado (fuera de sesión)
Removed:    entityManager.remove(u) — marcado para borrar
```

## Entidad básica

```java
@Entity
@Table(name = "usuarios")
public class Usuario {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "nombre", nullable = false, length = 100)
    private String nombre;

    @Column(name = "email", unique = true, nullable = false)
    private String email;

    @Column(name = "password_hash", nullable = false)
    private String passwordHash;

    @Column(name = "activo", nullable = false)
    private boolean activo = true;

    @CreationTimestamp
    @Column(name = "creado_en", updatable = false)
    private LocalDateTime creadoEn;

    @UpdateTimestamp
    @Column(name = "actualizado_en")
    private LocalDateTime actualizadoEn;
}
```

## GenerationType strategies

| Estrategia | Descripción |
|-----------|-------------|
| `IDENTITY` | BD genera el ID (AUTO_INCREMENT) — más común |
| `SEQUENCE` | Usa sequence de BD — mejor rendimiento en batch |
| `TABLE` | Tabla auxiliar para IDs — evitar |
| `UUID` | ID tipo UUID — buena para microservicios |

```java
// UUID
@Id
@GeneratedValue(strategy = GenerationType.UUID)
@Column(columnDefinition = "UUID")
private UUID id;
```

## @Embeddable — objetos de valor

```java
@Embeddable
public class Direccion {
    private String calle;
    private String ciudad;
    private String codigoPostal;
    private String pais;
}

@Entity
public class Cliente {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "calle", column = @Column(name = "dir_calle")),
        @AttributeOverride(name = "ciudad", column = @Column(name = "dir_ciudad"))
    })
    private Direccion direccion;
}
```

## Herencia de entidades

```java
// TABLE_PER_CLASS — tabla para cada clase
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo")
public abstract class Pago {
    @Id @GeneratedValue
    private Long id;
    private BigDecimal monto;
}

@Entity
@DiscriminatorValue("TARJETA")
public class PagoTarjeta extends Pago {
    private String numeroTarjeta;
}

@Entity
@DiscriminatorValue("EFECTIVO")
public class PagoEfectivo extends Pago {
    private String cajero;
}
```

## ⚠️ Errores Comunes

> [!warning] `spring.jpa.open-in-view=true` (default)
> Mantiene la sesión JPA abierta durante todo el request HTTP, lo que puede causar queries N+1 silenciosas y problemas de rendimiento. **Desactivar siempre en producción.**

> [!warning] `ddl-auto=create` o `update` en producción
> Nunca usar en prod. Usar `validate` o `none`. Las migraciones se hacen con Flyway/Liquibase.

## 📌 Notas Rápidas
