---
tags:
  - Java
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - Spring Boot
Palabras clave:
  - @Valid, @Validated, Bean Validation, @NotNull, @NotBlank, validación custom
---

## 🧠 Dependencia

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## Anotaciones de validación

```java
public record CrearUsuarioRequest(

    @NotNull(message = "El nombre es obligatorio")
    @NotBlank(message = "El nombre no puede estar vacío")
    @Size(min = 2, max = 50, message = "El nombre debe tener entre 2 y 50 caracteres")
    String nombre,

    @NotBlank
    @Email(message = "El email no es válido")
    String email,

    @NotBlank
    @Size(min = 8, message = "La contraseña debe tener mínimo 8 caracteres")
    @Pattern(regexp = ".*[A-Z].*", message = "La contraseña debe tener al menos una mayúscula")
    String password,

    @NotNull
    @Min(value = 18, message = "Debes ser mayor de edad")
    @Max(value = 120)
    Integer edad,

    @Past(message = "La fecha debe ser en el pasado")
    LocalDate fechaNacimiento,

    @Future(message = "La fecha debe ser en el futuro")
    LocalDate fechaVencimiento,

    @Positive(message = "El precio debe ser positivo")
    BigDecimal precio,

    @NotEmpty(message = "Debe tener al menos un elemento")
    List<String> roles
) {}
```

## Activar validación en el controller

```java
@PostMapping
public ResponseEntity<UsuarioResponse> crear(@Valid @RequestBody CrearUsuarioRequest request) {
    // Si falla la validación → 400 Bad Request automático
    return ResponseEntity.created(...).body(service.crear(request));
}

// Para @RequestParam y @PathVariable
@Validated // en la clase
@RestController
public class UsuarioController {
    @GetMapping("/{id}")
    public ResponseEntity<?> buscar(@PathVariable @Positive Long id) { ... }
}
```

## Validación custom — ConstraintValidator

```java
// 1. Crear la anotación
@Documented
@Constraint(validatedBy = UniqueEmailValidator.class)
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface UniqueEmail {
    String message() default "El email ya está en uso";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. Implementar el validador
@Component
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {

    @Autowired
    private UsuarioRepository repository;

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null) return true; // dejar a @NotNull manejarlo
        return !repository.existsByEmail(email);
    }
}

// 3. Usar
public record CrearUsuarioRequest(
    @UniqueEmail
    @Email
    String email
) {}
```

## Validación de grupos

```java
public interface CrearGroup {}
public interface ActualizarGroup {}

public class UsuarioRequest {
    @NotNull(groups = CrearGroup.class)
    private String password;

    @NotNull(groups = {CrearGroup.class, ActualizarGroup.class})
    private String nombre;
}

@PostMapping
public ResponseEntity<?> crear(@Validated(CrearGroup.class) @RequestBody UsuarioRequest req) { ... }

@PutMapping("/{id}")
public ResponseEntity<?> actualizar(@Validated(ActualizarGroup.class) @RequestBody UsuarioRequest req) { ... }
```

## Capturar errores de validación

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidacion(MethodArgumentNotValidException ex) {
        Map<String, String> errores = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errores.put(e.getField(), e.getDefaultMessage()));

        return ErrorResponse.builder()
            .codigo("VALIDATION_ERROR")
            .mensaje("Datos inválidos")
            .detalles(errores.entrySet().stream()
                .map(e -> e.getKey() + ": " + e.getValue())
                .toList())
            .build();
    }
}
```

## 📌 Notas Rápidas
- `@Valid` valida el objeto completo (incluye validaciones anidadas si el campo interno también tiene `@Valid`); `@Validated` (de Spring, no de Bean Validation) es necesario para validar `@RequestParam`/`@PathVariable` sueltos y para grupos de validación.
- Un `ConstraintValidator` custom puede inyectar dependencias (`@Autowired`) porque Spring lo gestiona como bean — útil para validaciones que necesitan consultar la base de datos (ej. email único).
- Los mensajes de `@NotBlank`/`@Email`/etc. se pueden externalizar a un `messages.properties` para soportar i18n en vez de hardcodearlos en la anotación.
- Validar en el DTO de entrada (borde de la API) no reemplaza validar invariantes de negocio dentro del dominio — son capas de protección distintas y complementarias.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

