---
tags:
  - Java
  - Spring
  - Swagger
  - OpenAPI
Creado: 2026-08-12
Relacionado:
  - Spring Boot
---

# Swagger y OpenAPI en Spring Boot

**OpenAPI** es la especificación (el estándar del documento que describe una API). **Swagger** es el conjunto de herramientas que la implementa (Swagger UI para visualizar, Swagger Codegen para generar clientes). En la práctica, "documentar con Swagger" = "generar y exponer el documento OpenAPI".

## Dependencia (springdoc-openapi, el estándar actual)

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

Con solo esta dependencia, Spring Boot ya expone:
- `/v3/api-docs` → el documento OpenAPI en JSON.
- `/swagger-ui.html` → la UI interactiva para probar los endpoints.

No hace falta configuración adicional para que empiece a documentar automáticamente los controllers existentes.

## Documentar un controller

```java
@RestController
@RequestMapping("/api/productos")
@Tag(name = "Productos", description = "Gestión de productos")
public class ProductoController {

    @Operation(summary = "Listar productos", description = "Devuelve todos los productos activos")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Listado exitoso"),
        @ApiResponse(responseCode = "500", description = "Error interno")
    })
    @GetMapping
    public List<ProductoDTO> listar() {
        return productoService.listar();
    }

    @Operation(summary = "Crear producto")
    @PostMapping
    public ResponseEntity<ProductoDTO> crear(@Valid @RequestBody ProductoDTO dto) {
        return ResponseEntity.status(HttpStatus.CREATED).body(productoService.crear(dto));
    }
}
```

## Documentar el DTO

```java
public record ProductoDTO(
    @Schema(description = "ID del producto", example = "1")
    Long id,

    @Schema(description = "Nombre del producto", example = "Teclado mecánico", required = true)
    @NotBlank
    String nombre,

    @Schema(description = "Precio en soles", example = "150.00")
    @Positive
    BigDecimal precio
) {}
```

Bean Validation (`@NotBlank`, `@Positive`) se refleja automáticamente en el schema generado — no hace falta duplicar la información.

## Configuración global (info de la API)

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("API de Productos")
                .version("1.0")
                .description("API para gestión de catálogo")
                .contact(new Contact().name("Equipo Backend").email("backend@empresa.com")));
    }
}
```

O vía `application.properties`:
```properties
springdoc.swagger-ui.path=/docs
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.operationsSorter=method
```

## Documentar seguridad (JWT/OAuth2) en el Swagger UI

```java
@Configuration
@SecurityScheme(
    name = "bearerAuth",
    type = SecuritySchemeType.HTTP,
    scheme = "bearer",
    bearerFormat = "JWT"
)
public class OpenApiSecurityConfig {}
```

```java
@Operation(summary = "Endpoint protegido")
@SecurityRequirement(name = "bearerAuth")
@GetMapping("/perfil")
public UsuarioDTO perfil() { /* ... */ }
```

Esto agrega un botón "Authorize" en Swagger UI donde se pega el token — clave para poder probar endpoints protegidos desde la misma UI sin herramientas externas.

## Agrupar APIs (útil si hay endpoints públicos y de admin)

```java
@Bean
public GroupedOpenApi publicApi() {
    return GroupedOpenApi.builder()
        .group("publica")
        .pathsToMatch("/api/productos/**")
        .build();
}

@Bean
public GroupedOpenApi adminApi() {
    return GroupedOpenApi.builder()
        .group("admin")
        .pathsToMatch("/api/admin/**")
        .build();
}
```

## Buenas prácticas

- No exponer `swagger-ui` en producción sin protección (o directamente deshabilitarlo con un profile) — es un mapa completo de la API para un atacante.
```properties
%prod.springdoc.swagger-ui.enabled=false
%prod.springdoc.api-docs.enabled=false
```
- Usar `@Schema(example = ...)` en los DTOs para que la documentación sea usable sin adivinar formatos.
- Mantener las anotaciones `@Operation`/`@ApiResponses` actualizadas — documentación desactualizada es peor que no tener documentación (genera desconfianza).

## Errores comunes

- Dejar Swagger UI accesible públicamente en producción.
- No documentar los códigos de error posibles (`@ApiResponses`), dejando al consumidor de la API adivinando qué puede fallar.
- Duplicar descripciones entre el código y un Postman/documento aparte, que se desincroniza — el punto de springdoc es que el documento **se genera del código real**, siempre actualizado.

## Checklist
- [ ] Tengo Swagger UI funcionando en un proyecto Spring Boot con springdoc-openapi.
- [ ] Documento controllers con `@Operation` y DTOs con `@Schema`.
- [ ] Sé configurar el botón de autenticación (bearerAuth) para probar endpoints protegidos.
- [ ] Deshabilito Swagger UI en producción o lo protejo adecuadamente.
