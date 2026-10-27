---
tags:
  - Java
  - Spring
  - JPA
Creado: 2026-05-21
Relacionado:
  - Spring Data
Palabras clave:
  - JpaRepository, CrudRepository, derived queries, custom queries, Pageable
---

## 🧠 Jerarquía de repositorios

```
Repository (interfaz base)
    └── CrudRepository<T, ID>    → CRUD básico
        └── PagingAndSortingRepository<T, ID>  → + paginación
            └── JpaRepository<T, ID>           → + JPA específico ✅ usar este
```

## JpaRepository — métodos disponibles

```java
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    // Ya tienes sin escribir nada:
}

// Uso
repository.save(usuario);           // INSERT o UPDATE
repository.saveAll(listaUsuarios);  // batch save
repository.findById(1L);            // Optional<Usuario>
repository.findAll();               // List<Usuario>
repository.findAll(Sort.by("nombre"));
repository.findAll(pageable);       // Page<Usuario>
repository.existsById(1L);          // boolean
repository.count();                 // long
repository.deleteById(1L);
repository.delete(usuario);
repository.deleteAll(lista);
repository.flush();                 // forzar sync con BD
repository.saveAndFlush(usuario);
```

## Derived Query Methods (queries por nombre de método)

```java
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {

    // findBy, getBy, readBy, queryBy, countBy, existsBy, deleteBy
    Optional<Usuario> findByEmail(String email);
    List<Usuario> findByNombre(String nombre);
    List<Usuario> findByActivoTrue();
    List<Usuario> findByEdadGreaterThan(int edad);
    List<Usuario> findByEdadBetween(int min, int max);
    List<Usuario> findByNombreContainingIgnoreCase(String nombre);
    List<Usuario> findByNombreStartingWith(String prefijo);
    List<Usuario> findByRol(Rol rol);

    // Múltiples condiciones
    List<Usuario> findByActivoTrueAndRol(Rol rol);
    Optional<Usuario> findByEmailAndActivo(String email, boolean activo);

    // Con ordenamiento
    List<Usuario> findByActivoTrueOrderByNombreAsc();

    // Con paginación
    Page<Usuario> findByActivo(boolean activo, Pageable pageable);

    // Contar y existir
    long countByRol(Rol rol);
    boolean existsByEmail(String email);

    // Top/First
    Optional<Usuario> findFirstByOrderByCreadoEnDesc();
    List<Usuario> findTop5ByActivoTrueOrderByCreadoEnDesc();
}
```

## @Query — JPQL personalizado

```java
// JPQL (lenguaje JPA, trabaja con entidades, no tablas)
@Query("SELECT u FROM Usuario u WHERE u.email = :email AND u.activo = true")
Optional<Usuario> findByEmailActivo(@Param("email") String email);

// JOIN FETCH para evitar N+1
@Query("SELECT u FROM Usuario u LEFT JOIN FETCH u.pedidos WHERE u.id = :id")
Optional<Usuario> findByIdWithPedidos(@Param("id") Long id);

// Con paginación
@Query("SELECT u FROM Usuario u WHERE u.rol = :rol")
Page<Usuario> findByRol(@Param("rol") Rol rol, Pageable pageable);

// Native SQL
@Query(value = "SELECT * FROM usuarios WHERE LOWER(email) = LOWER(:email)",
       nativeQuery = true)
Optional<Usuario> findByEmailNative(@Param("email") String email);
```

## Projections — traer solo ciertos campos

```java
// Interface projection
public interface UsuarioResumen {
    Long getId();
    String getNombre();
    String getEmail();
}

public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    List<UsuarioResumen> findByActivo(boolean activo);
}

// Class-based (DTO) projection
public record UsuarioDTO(Long id, String nombre) {}

@Query("SELECT new com.juber.dto.UsuarioDTO(u.id, u.nombre) FROM Usuario u")
List<UsuarioDTO> findAllDTO();
```

## @Modifying — UPDATE y DELETE con @Query

```java
@Modifying
@Query("UPDATE Usuario u SET u.activo = false WHERE u.id = :id")
int desactivarUsuario(@Param("id") Long id);

@Modifying
@Query("DELETE FROM Usuario u WHERE u.activo = false AND u.creadoEn < :fecha")
int eliminarInactivos(@Param("fecha") LocalDateTime fecha);

// ¡Siempre dentro de @Transactional!
@Transactional
public void desactivar(Long id) {
    repository.desactivarUsuario(id);
}
```

## 📌 Notas Rápidas
- `JpaRepository` es la implementación de Spring del **Repository pattern** (originalmente descrito en Domain-Driven Design) — abstrae la persistencia detrás de una interfaz, el dominio no sabe si detrás hay JPA, MongoDB o un mock.
- Los "Derived Query Methods" generan la query a partir del nombre del método en tiempo de arranque — si el nombre no coincide con un campo real de la entidad, falla al iniciar la app (fail-fast), no en runtime al llamarlo.
- Para queries complejas dinámicas (filtros opcionales combinables), Specifications es más mantenible que encadenar muchos métodos derivados o construir JPQL con concatenación de strings.
- `@Modifying` + `@Query` siempre requiere `@Transactional` en el método que lo llama — sin eso, Spring Data lanza `InvalidDataAccessApiUsageException`.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

