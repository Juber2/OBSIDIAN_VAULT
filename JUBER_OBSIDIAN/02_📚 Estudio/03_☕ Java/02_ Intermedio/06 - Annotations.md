---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
Palabras clave:
  - annotations, anotaciones, @Override, @Deprecated, meta-anotaciones, custom annotations
---

## 🧠 Concepto

Las **anotaciones** son metadatos que se añaden al código para proporcionar información al compilador, frameworks o herramientas de procesamiento. No cambian la lógica del programa directamente.

## Anotaciones estándar de Java

```java
// @Override — verificar que se está sobreescribiendo un método del padre
@Override
public String toString() { return "..."; }

// @Deprecated — marcar como obsoleto
@Deprecated(since = "2.0", forRemoval = true)
public void metodoViejo() { ... }

// @SuppressWarnings — suprimir advertencias del compilador
@SuppressWarnings("unchecked")
public void metodo() { ... }

// @FunctionalInterface — verificar que es interfaz funcional
@FunctionalInterface
public interface Calculadora {
    int operar(int a, int b);
}

// @SafeVarargs — varargs genéricos seguros
@SafeVarargs
public final <T> List<T> asList(T... elementos) { ... }
```

## Anotaciones de uso común en frameworks

```java
// Spring / Spring Boot
@Component, @Service, @Repository, @Controller, @RestController
@Autowired, @Qualifier, @Value
@RequestMapping, @GetMapping, @PostMapping
@Transactional
@Bean, @Configuration

// JPA / Hibernate
@Entity, @Table, @Id, @GeneratedValue
@Column, @ManyToOne, @OneToMany, @JoinColumn

// Validaciones (Jakarta Bean Validation)
@NotNull, @NotBlank, @Size, @Min, @Max, @Email, @Pattern

// Lombok
@Data, @Builder, @NoArgsConstructor, @AllArgsConstructor, @Getter, @Setter
```

## Crear anotaciones personalizadas

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)  // disponible en runtime
@Target(ElementType.METHOD)          // solo en métodos
@Documented                          // incluir en JavaDoc
public @interface Auditable {
    String descripcion() default "";
    boolean registrarParametros() default true;
}

// Uso
@Auditable(descripcion = "Crear usuario", registrarParametros = true)
public void crearUsuario(Usuario usuario) { ... }
```

## Meta-anotaciones

Anotaciones que se aplican a otras anotaciones.

| Meta-anotación | Descripción |
|---------------|-------------|
| `@Retention` | Cuánto tiempo vive la anotación |
| `@Target` | Dónde se puede aplicar |
| `@Documented` | Incluir en JavaDoc |
| `@Inherited` | Subclases heredan la anotación |
| `@Repeatable` | Puede aplicarse múltiples veces |

### `@Retention` valores

| Valor | Descripción |
|-------|-------------|
| `SOURCE` | Solo en código fuente, el compilador la descarta |
| `CLASS` | En el .class pero no en runtime (default) |
| `RUNTIME` | Disponible en runtime vía Reflection |

### `@Target` valores

| Valor | Dónde aplica |
|-------|-------------|
| `TYPE` | Clases, interfaces, enums |
| `METHOD` | Métodos |
| `FIELD` | Campos |
| `PARAMETER` | Parámetros de métodos |
| `CONSTRUCTOR` | Constructores |
| `LOCAL_VARIABLE` | Variables locales |

## Leer anotaciones con Reflection

```java
Method metodo = MiClase.class.getMethod("crearUsuario", Usuario.class);

if (metodo.isAnnotationPresent(Auditable.class)) {
    Auditable auditable = metodo.getAnnotation(Auditable.class);
    System.out.println("Descripción: " + auditable.descripcion());
}
```

## ⚠️ Errores Comunes

> [!warning] Olvidar `@Retention(RUNTIME)` en anotaciones custom
> Sin esto, la anotación no estará disponible en runtime y no podrás leerla con Reflection.

## 📌 Notas Rápidas
- `@Retention` define hasta cuándo "vive" la anotación: `SOURCE` (solo compilación), `CLASS` (en el .class, no en runtime), `RUNTIME` (accesible por Reflection).
- `@Target` restringe dónde se puede aplicar (`TYPE`, `METHOD`, `FIELD`, etc.) — evita usos incorrectos en compile-time.
- Frameworks como Spring/Hibernate dependen fuertemente de anotaciones `RUNTIME` + Reflection para su "magia" (DI, mapeo ORM).
- Las meta-anotaciones (anotaciones sobre anotaciones) permiten crear anotaciones compuestas, como hace Spring con `@RestController` (que combina `@Controller` + `@ResponseBody`).

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

