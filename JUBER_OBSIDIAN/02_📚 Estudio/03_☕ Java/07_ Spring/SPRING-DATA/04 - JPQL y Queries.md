---
tags:
  - Java
  - Spring
  - JPA
Creado: 2026-05-21
Relacionado:
  - Spring Data
  - Repositorios
Palabras clave:
  - JPQL, Criteria API, Specification, QueryDSL, named queries
---

## 🧠 JPQL vs SQL

```sql
-- SQL (trabaja con tablas y columnas)
SELECT u.nombre, u.email FROM usuarios u WHERE u.activo = true

-- JPQL (trabaja con entidades y campos Java)
SELECT u.nombre, u.email FROM Usuario u WHERE u.activo = true
--     ^^^^^^^^ nombre de la clase Java, no la tabla
```

## JPQL — sintaxis completa

```java
// SELECT
"SELECT u FROM Usuario u"
"SELECT u.nombre FROM Usuario u"
"SELECT u.nombre, u.email FROM Usuario u"  // Object[]

// WHERE
"SELECT u FROM Usuario u WHERE u.activo = true"
"SELECT u FROM Usuario u WHERE u.edad > :minEdad"
"SELECT u FROM Usuario u WHERE u.nombre LIKE :patron"
"SELECT u FROM Usuario u WHERE u.nombre IN :nombres"
"SELECT u FROM Usuario u WHERE u.nombre IS NULL"

// JOIN
"SELECT p FROM Pedido p JOIN p.usuario u WHERE u.id = :usuarioId"
"SELECT p FROM Pedido p JOIN FETCH p.usuario"  // JOIN FETCH carga la relación
"SELECT p FROM Pedido p LEFT JOIN p.envio e WHERE e IS NULL"

// ORDER BY
"SELECT u FROM Usuario u ORDER BY u.nombre ASC, u.creadoEn DESC"

// GROUP BY / HAVING
"SELECT u.rol, COUNT(u) FROM Usuario u GROUP BY u.rol HAVING COUNT(u) > 5"

// Funciones
"SELECT UPPER(u.nombre) FROM Usuario u"
"SELECT CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP FROM Usuario u"
"SELECT LENGTH(u.nombre) FROM Usuario u"
"SELECT CONCAT(u.nombre, ' ', u.apellido) FROM Usuario u"
```

## Criteria API — queries dinámicas type-safe

```java
@Service
public class UsuarioBusquedaService {

    @PersistenceContext
    private EntityManager em;

    public List<Usuario> buscar(String nombre, Rol rol, Boolean activo) {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Usuario> query = cb.createQuery(Usuario.class);
        Root<Usuario> root = query.from(Usuario.class);

        List<Predicate> predicates = new ArrayList<>();

        if (nombre != null && !nombre.isBlank()) {
            predicates.add(cb.like(cb.lower(root.get("nombre")),
                "%" + nombre.toLowerCase() + "%"));
        }
        if (rol != null) {
            predicates.add(cb.equal(root.get("rol"), rol));
        }
        if (activo != null) {
            predicates.add(cb.equal(root.get("activo"), activo));
        }

        query.where(predicates.toArray(new Predicate[0]));
        query.orderBy(cb.asc(root.get("nombre")));

        return em.createQuery(query).getResultList();
    }
}
```

## Specifications — Criteria API más limpio

```java
// Specification<T> — un predicado reutilizable
public class UsuarioSpec {

    public static Specification<Usuario> tieneNombre(String nombre) {
        return (root, query, cb) -> nombre == null ? null :
            cb.like(cb.lower(root.get("nombre")), "%" + nombre.toLowerCase() + "%");
    }

    public static Specification<Usuario> tieneRol(Rol rol) {
        return (root, query, cb) -> rol == null ? null :
            cb.equal(root.get("rol"), rol);
    }

    public static Specification<Usuario> estaActivo(Boolean activo) {
        return (root, query, cb) -> activo == null ? null :
            cb.equal(root.get("activo"), activo);
    }
}

// Repository debe extender JpaSpecificationExecutor
public interface UsuarioRepository extends JpaRepository<Usuario, Long>,
                                            JpaSpecificationExecutor<Usuario> {}

// Uso — composición fluent
List<Usuario> usuarios = repository.findAll(
    where(tieneNombre("Juber"))
    .and(tieneRol(Rol.ADMIN))
    .and(estaActivo(true))
);
```

## Named Queries

```java
@Entity
@NamedQuery(
    name = "Usuario.findByEmailActivo",
    query = "SELECT u FROM Usuario u WHERE u.email = :email AND u.activo = true"
)
public class Usuario { ... }

// En el repositorio
@Query(name = "Usuario.findByEmailActivo")
Optional<Usuario> findByEmailActivo(@Param("email") String email);
```

## 📌 Notas Rápidas
