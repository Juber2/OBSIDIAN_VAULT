---
tags:
  - Java
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - Spring Boot
  - REST Controllers
Palabras clave:
  - DTO, record, Jackson, serialización, MapStruct, @JsonProperty, paginación
---

## 🧠 DTOs (Data Transfer Objects)

Nunca exponer entidades directamente. Usar DTOs para entrada y salida.

```java
// Request DTO (entrada)
public record CrearUsuarioRequest(
    @NotBlank String nombre,
    @Email @NotBlank String email,
    @Size(min = 8) String password
) {}

// Response DTO (salida) — sin password, sin campos internos
public record UsuarioResponse(
    Long id,
    String nombre,
    String email,
    LocalDateTime creadoEn
) {
    public static UsuarioResponse from(Usuario usuario) {
        return new UsuarioResponse(
            usuario.getId(),
            usuario.getNombre(),
            usuario.getEmail(),
            usuario.getCreadoEn()
        );
    }
}
```

## MapStruct — mapeo automático DTO ↔ Entidad

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>
```

```java
@Mapper(componentModel = "spring")
public interface UsuarioMapper {

    UsuarioResponse toResponse(Usuario usuario);

    // Si los nombres no coinciden
    @Mapping(source = "nombre", target = "nombreCompleto")
    @Mapping(source = "createdAt", target = "creadoEn")
    UsuarioResponse toResponseCustom(Usuario usuario);

    Usuario toEntity(CrearUsuarioRequest request);

    List<UsuarioResponse> toResponseList(List<Usuario> usuarios);
}

// Uso
@Autowired
private UsuarioMapper mapper;

UsuarioResponse response = mapper.toResponse(usuario);
```

## Jackson — configuración JSON

```java
// application.properties
spring.jackson.default-property-inclusion=non_null    // omitir nulls
spring.jackson.serialization.write-dates-as-timestamps=false  // fechas ISO
spring.jackson.date-format=yyyy-MM-dd'T'HH:mm:ss

// O con @Bean
@Bean
public Jackson2ObjectMapperBuilderCustomizer jsonCustomizer() {
    return builder -> builder
        .serializationInclusion(JsonInclude.Include.NON_NULL)
        .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
        .modules(new JavaTimeModule());
}
```

## Anotaciones Jackson útiles

```java
public class UsuarioResponse {
    @JsonProperty("usuario_id")    // nombre custom en JSON
    private Long id;

    @JsonIgnore                    // no incluir en JSON
    private String password;

    @JsonFormat(pattern = "dd/MM/yyyy HH:mm")
    private LocalDateTime creadoEn;

    @JsonInclude(JsonInclude.Include.NON_NULL) // omitir si null
    private String telefono;
}
```

## Paginación

```java
// Controller
@GetMapping
public ResponseEntity<Page<UsuarioResponse>> listar(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "id") String sortBy,
    @RequestParam(defaultValue = "asc") String direction
) {
    Sort.Direction dir = Sort.Direction.fromString(direction);
    Pageable pageable = PageRequest.of(page, size, Sort.by(dir, sortBy));
    return ResponseEntity.ok(service.listar(pageable));
}

// Response incluye: content[], totalElements, totalPages, number, size
// GET /api/usuarios?page=0&size=10&sortBy=nombre&direction=desc
```

## Error Response estándar

```java
@Getter
@Builder
public class ErrorResponse {
    private final String codigo;
    private final String mensaje;
    private final List<String> detalles;
    private final LocalDateTime timestamp = LocalDateTime.now();
    private final String path;
}

// Uso en el handler global
@ExceptionHandler(NotFoundException.class)
@ResponseStatus(HttpStatus.NOT_FOUND)
public ErrorResponse handleNotFound(NotFoundException ex, HttpServletRequest request) {
    return ErrorResponse.builder()
        .codigo("NOT_FOUND")
        .mensaje(ex.getMessage())
        .path(request.getRequestURI())
        .build();
}
```

## 📌 Notas Rápidas
