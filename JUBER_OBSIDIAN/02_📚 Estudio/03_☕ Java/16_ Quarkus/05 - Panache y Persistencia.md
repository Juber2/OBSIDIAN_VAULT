---
tags:
  - Java
  - Quarkus
Creado: 2026-08-09
Relacionado:
  - Quarkus
---

# Panache y Persistencia

**Panache** es la capa de Quarkus sobre Hibernate ORM que elimina boilerplate de repositorios — pensado para reducir lo que en Spring Data JPA todavía requiere escribir interfaces `JpaRepository`.

## Extension necesaria

```bash
mvn quarkus:add-extension -Dextensions="hibernate-orm-panache,jdbc-postgresql"
```

```properties
# application.properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=postgres
quarkus.datasource.password=postgres
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/midb
quarkus.hibernate-orm.database.generation=update
```

## Estilo Active Record (entidad con métodos propios)

```java
@Entity
public class Producto extends PanacheEntity {
    public String nombre;
    public BigDecimal precio;
    public String categoria;

    public static List<Producto> porCategoria(String categoria) {
        return list("categoria", categoria);
    }
}
```

Uso directo, sin repositorio intermedio:
```java
Producto producto = new Producto();
producto.nombre = "Teclado";
producto.precio = new BigDecimal("50.00");
producto.persist();

List<Producto> todos = Producto.listAll();
Producto uno = Producto.findById(1L);
long total = Producto.count();
Producto.deleteById(1L);
```

`PanacheEntity` ya trae un `id` de tipo `Long` autogenerado. Si necesitás un tipo de ID distinto o control total, se usa `PanacheEntityBase` en su lugar.

## Estilo Repository (para quien prefiere separar persistencia de la entidad)

```java
@Entity
public class Producto {
    @Id @GeneratedValue
    public Long id;
    public String nombre;
    public BigDecimal precio;
}

@ApplicationScoped
public class ProductoRepository implements PanacheRepository<Producto> {
    public List<Producto> porCategoria(String categoria) {
        return list("categoria", categoria);
    }
}
```

```java
@Inject
ProductoRepository productoRepository;

List<Producto> productos = productoRepository.listAll();
```

Este estilo es más parecido a `JpaRepository` de Spring Data y más fácil de testear con mocks.

## Queries

```java
// Named/positional params
List<Producto> baratos = Producto.list("precio < ?1", new BigDecimal("100"));

// Named params
List<Producto> porCat = Producto.list("categoria = :cat", Parameters.with("cat", "electronica"));

// Paginación
List<Producto> pagina = Producto.findAll().page(Page.of(0, 20)).list();

// Ordenamiento
List<Producto> ordenados = Producto.listAll(Sort.by("precio").descending());
```

## Transacciones

```java
@ApplicationScoped
public class PedidoService {

    @Transactional
    public void crearPedidoConItems(Pedido pedido, List<Item> items) {
        pedido.persist();
        items.forEach(item -> {
            item.pedidoId = pedido.id;
            item.persist();
        });
    }
}
```

`@Transactional` de Jakarta (no Spring) — mismo concepto: si algo lanza excepción runtime dentro, se hace rollback.

## Migraciones con Flyway

```bash
mvn quarkus:add-extension -Dextensions="flyway"
```

```properties
quarkus.flyway.migrate-at-start=true
```

Los scripts van en `src/main/resources/db/migration/V1__init.sql` — igual convención que Flyway en cualquier otro stack.

## Errores comunes

- Elegir Active Record (`PanacheEntity`) para dominios complejos donde la entidad termina mezclando lógica de persistencia con lógica de negocio — para dominios ricos, el estilo Repository separa mejor las responsabilidades.
- No usar `@Transactional` en operaciones multi-paso, dejando estado inconsistente si falla a mitad de camino.
- Olvidar `migrate-at-start=false` en producción y dejar que Flyway corra automáticamente sin control — mejor correr migraciones como paso explícito de deploy en ambientes críticos.

## Checklist
- [ ] Sé elegir entre estilo Active Record y Repository según el caso.
- [ ] Sé escribir queries con Panache (`list`, `find`, paginación, ordenamiento).
- [ ] Uso `@Transactional` correctamente en operaciones multi-paso.
- [ ] Sé configurar migraciones con Flyway en Quarkus.
