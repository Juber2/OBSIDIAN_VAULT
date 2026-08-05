---
tags:
  - Java
  - Spring
  - JPA
Creado: 2026-05-21
Relacionado:
  - Spring Data
  - JPA e Hibernate
Palabras clave:
  - @OneToMany, @ManyToOne, @ManyToMany, @OneToOne, FetchType, CascadeType, N+1
---

## Relaciones JPA

### @ManyToOne — muchos a uno (más común)

```java
@Entity
public class Pedido {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY) // SIEMPRE LAZY en @ManyToOne
    @JoinColumn(name = "usuario_id", nullable = false)
    private Usuario usuario;

    private BigDecimal total;
}
```

### @OneToMany — uno a muchos

```java
@Entity
public class Usuario {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "usuario",            // campo en el lado "Many"
               cascade = CascadeType.ALL,        // operaciones se propagan
               orphanRemoval = true,             // borrar huérfanos
               fetch = FetchType.LAZY)           // SIEMPRE LAZY
    private List<Pedido> pedidos = new ArrayList<>();

    // Métodos helper para mantener coherencia bidireccional
    public void agregarPedido(Pedido pedido) {
        pedidos.add(pedido);
        pedido.setUsuario(this);
    }

    public void removerPedido(Pedido pedido) {
        pedidos.remove(pedido);
        pedido.setUsuario(null);
    }
}
```

### @ManyToMany — muchos a muchos

```java
@Entity
public class Estudiante {
    @ManyToMany
    @JoinTable(
        name = "estudiante_curso",
        joinColumns = @JoinColumn(name = "estudiante_id"),
        inverseJoinColumns = @JoinColumn(name = "curso_id")
    )
    private Set<Curso> cursos = new HashSet<>();
}

@Entity
public class Curso {
    @ManyToMany(mappedBy = "cursos")
    private Set<Estudiante> estudiantes = new HashSet<>();
}
```

### @OneToOne

```java
@Entity
public class Usuario {
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "perfil_id", referencedColumnName = "id")
    private Perfil perfil;
}
```

## FetchType

| Tipo | Descripción | Recomendación |
|------|-------------|---------------|
| `LAZY` | Carga la relación solo cuando se accede | ✅ Default para todos |
| `EAGER` | Carga siempre junto con la entidad padre | ❌ Evitar (N+1, over-fetching) |

> [!important] Defaults de JPA:
> - `@ManyToOne` y `@OneToOne` → EAGER por defecto → **cambiar siempre a LAZY**
> - `@OneToMany` y `@ManyToMany` → LAZY por defecto → bien

## CascadeType

| Tipo | Qué propaga |
|------|-------------|
| `PERSIST` | Al guardar el padre, guarda los hijos |
| `MERGE` | Al actualizar el padre, actualiza los hijos |
| `REMOVE` | Al borrar el padre, borra los hijos |
| `REFRESH` | Al refrescar el padre, refresca los hijos |
| `DETACH` | Al despegar el padre, despega los hijos |
| `ALL` | Todos los anteriores |

## Problema N+1 — el más importante a entender

```java
// ❌ N+1: una query para obtener los pedidos + una por cada usuario
List<Pedido> pedidos = pedidoRepository.findAll();
pedidos.forEach(p -> System.out.println(p.getUsuario().getNombre())); // N queries

// ✅ JOIN FETCH: una sola query
@Query("SELECT p FROM Pedido p JOIN FETCH p.usuario")
List<Pedido> findAllWithUsuario();

// ✅ EntityGraph: declarativo
@EntityGraph(attributePaths = {"usuario"})
List<Pedido> findAll();
```

## equals() y hashCode() en entidades

```java
@Entity
public class Usuario {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Basarse en ID para entidades persistidas, o en campo de negocio único
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Usuario)) return false;
        Usuario u = (Usuario) o;
        return id != null && id.equals(u.id);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode(); // constante para entidades Hibernate
    }
}
```

## 📌 Notas Rápidas
