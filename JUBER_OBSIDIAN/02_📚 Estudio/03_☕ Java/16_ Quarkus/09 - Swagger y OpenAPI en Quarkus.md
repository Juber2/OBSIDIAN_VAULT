---
tags:
  - Java
  - Quarkus
  - Swagger
  - OpenAPI
Creado: 2026-08-12
Relacionado:
  - Quarkus
---

# Swagger y OpenAPI en Quarkus

Quarkus usa la extension `quarkus-smallrye-openapi`, que implementa el estándar **MicroProfile OpenAPI** — mismo resultado que springdoc en Spring, distinta implementación por debajo (Quarkus usa anotaciones estándar Jakarta/Eclipse MicroProfile, no las de springdoc).

## Extension

```bash
mvn quarkus:add-extension -Dextensions="smallrye-openapi"
```

Con esto, quedan disponibles automáticamente:
- `/q/openapi` → documento OpenAPI en YAML/JSON.
- `/q/swagger-ui` → UI interactiva (equivalente a `/swagger-ui.html` en Spring).

## Documentar un endpoint

```java
@Path("/api/productos")
@Tag(name = "Productos", description = "Gestión de productos")
public class ProductoResource {

    @GET
    @Operation(summary = "Listar productos", description = "Devuelve todos los productos activos")
    @APIResponse(responseCode = "200", description = "Listado exitoso")
    public List<ProductoDTO> listar() {
        return productoService.listar();
    }

    @POST
    @Operation(summary = "Crear producto")
    @APIResponse(responseCode = "201", description = "Producto creado")
    public Response crear(@Valid ProductoDTO dto) {
        var creado = productoService.crear(dto);
        return Response.status(201).entity(creado).build();
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

Igual que en Spring: las anotaciones de Bean Validation ya aportan información al schema generado automáticamente.

## Configuración

```properties
quarkus.swagger-ui.path=/docs
quarkus.smallrye-openapi.path=/openapi

mp.openapi.extensions.smallrye.info.title=API de Productos
mp.openapi.extensions.smallrye.info.version=1.0.0
```

O programáticamente:
```java
@OpenAPIDefinition(
    info = @Info(
        title = "API de Productos",
        version = "1.0",
        contact = @Contact(name = "Equipo Backend", email = "backend@empresa.com")
    )
)
public class OpenApiConfig extends Application {}
```

## Seguridad (JWT) en la documentación

```java
@SecurityScheme(
    securitySchemeName = "bearerAuth",
    type = SecuritySchemeType.HTTP,
    scheme = "bearer",
    bearerFormat = "JWT"
)
public class OpenApiSecurityConfig {}
```

```java
@GET
@Path("/perfil")
@SecurityRequirement(name = "bearerAuth")
public UsuarioDTO perfil() { /* ... */ }
```

## Habilitar/deshabilitar por ambiente

```properties
%prod.quarkus.swagger-ui.always-include=false
```

Por default, `swagger-ui` solo está disponible en Dev Mode — en producción hay que habilitarlo explícitamente si se necesita (`always-include=true`), lo cual ya es más seguro por default que el comportamiento típico de Spring Boot (donde hay que acordarse de deshabilitarlo).

## Tabla comparativa rápida con Spring

| Concepto | Spring Boot (springdoc) | Quarkus (smallrye-openapi) |
|---|---|---|
| Estándar de anotaciones | springdoc / Swagger annotations | MicroProfile OpenAPI (Jakarta) |
| UI | `/swagger-ui.html` | `/q/swagger-ui` |
| Documento OpenAPI | `/v3/api-docs` | `/q/openapi` |
| Default en producción | habilitado (hay que apagarlo a mano) | deshabilitado por default fuera de dev |

## Errores comunes

- Asumir que las anotaciones son 100% idénticas a Spring — la mayoría de nombres coinciden (`@Operation`, `@Tag`, `@Schema`) porque ambos siguen el estándar OpenAPI, pero el paquete de origen (`org.eclipse.microprofile.openapi` en Quarkus) es distinto al de springdoc.
- No configurar `@SecurityScheme` y no poder probar endpoints protegidos desde la UI.
- No revisar el comportamiento por-ambiente y exponer swagger-ui sin querer en producción (o al revés, olvidarlo deshabilitado en un ambiente donde sí se necesita para pruebas).

## Checklist
- [ ] Tengo Swagger UI funcionando en un proyecto Quarkus con smallrye-openapi.
- [ ] Documento resources con `@Operation`, `@Tag`, `@APIResponse`.
- [ ] Sé configurar seguridad JWT en la documentación.
- [ ] Entiendo la diferencia de comportamiento por-ambiente entre Quarkus y Spring Boot en este tema.
- [ ] Puedo explicar en entrevista la equivalencia conceptual entre ambos enfoques (mismo estándar OpenAPI, distinta implementación).

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

