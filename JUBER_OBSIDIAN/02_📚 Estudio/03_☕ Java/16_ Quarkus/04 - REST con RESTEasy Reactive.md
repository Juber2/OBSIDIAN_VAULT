---
tags:
  - Java
  - Quarkus
Creado: 2026-08-09
Relacionado:
  - Quarkus
---

# REST con RESTEasy Reactive

Quarkus usa **JAX-RS** (estándar Jakarta) como base para REST, implementado por RESTEasy Reactive — no Spring MVC. La sintaxis de anotaciones difiere de `@RestController`/`@GetMapping`, aunque el concepto es el mismo.

## Endpoint básico

```java
@Path("/api/productos")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ProductoResource {

    @Inject
    ProductoService productoService;

    @GET
    public List<ProductoDTO> listar() {
        return productoService.listar();
    }

    @GET
    @Path("/{id}")
    public ProductoDTO obtener(@PathParam("id") Long id) {
        return productoService.buscarPorId(id);
    }

    @POST
    public Response crear(@Valid ProductoDTO dto) {
        ProductoDTO creado = productoService.crear(dto);
        return Response.status(Response.Status.CREATED).entity(creado).build();
    }

    @PUT
    @Path("/{id}")
    public ProductoDTO actualizar(@PathParam("id") Long id, @Valid ProductoDTO dto) {
        return productoService.actualizar(id, dto);
    }

    @DELETE
    @Path("/{id}")
    public Response eliminar(@PathParam("id") Long id) {
        productoService.eliminar(id);
        return Response.noContent().build();
    }
}
```

## Tabla comparativa de anotaciones (para quien viene de Spring)

| Spring MVC | JAX-RS (Quarkus) |
|---|---|
| `@RestController` | `@Path` a nivel clase |
| `@GetMapping` | `@GET` + `@Path` |
| `@PostMapping` | `@POST` + `@Path` |
| `@PathVariable` | `@PathParam` |
| `@RequestParam` | `@QueryParam` |
| `@RequestBody` | parámetro directo (sin anotación) |
| `@RequestHeader` | `@HeaderParam` |

## Query params y validación

```java
@GET
public List<ProductoDTO> buscar(
        @QueryParam("categoria") String categoria,
        @QueryParam("page") @DefaultValue("0") int page) {
    return productoService.buscar(categoria, page);
}
```

```java
public record ProductoDTO(
    Long id,
    @NotBlank String nombre,
    @Positive BigDecimal precio
) {}
```

Bean Validation (`@Valid`, `@NotBlank`, etc.) funciona igual que en Spring — es el mismo estándar Jakarta Bean Validation.

## Manejo de errores centralizado

```java
@Provider
public class NotFoundExceptionMapper implements ExceptionMapper<EntityNotFoundException> {
    @Override
    public Response toResponse(EntityNotFoundException e) {
        return Response.status(Response.Status.NOT_FOUND)
                .entity(new ErrorResponse(e.getMessage()))
                .build();
    }
}
```

Equivalente a `@ExceptionHandler`/`@ControllerAdvice` de Spring.

## Reactivo (Mutiny) — la diferencia real con Spring MVC clásico

RESTEasy **Reactive** soporta devolver tipos reactivos (`Uni<T>`, `Multi<T>` de Mutiny) sin bloquear threads:

```java
@GET
@Path("/{id}")
public Uni<ProductoDTO> obtener(@PathParam("id") Long id) {
    return productoService.buscarPorIdReactivo(id);
}
```

No es obligatorio usar reactivo — un método que devuelve `ProductoDTO` directo (imperativo) funciona igual de bien y es más simple de entender al empezar. Reactivo se justifica cuando hay mucha I/O concurrente (llamadas a otros servicios, DB) y se busca throughput alto sin escalar threads 1:1.

## Errores comunes

- Mezclar estilo imperativo y reactivo sin entender que bloquear dentro de un método reactivo (`Uni`/`Multi`) anula el beneficio — hay que usar `@Blocking` explícitamente si un método reactivo necesita hacer algo bloqueante.
- Confundir `@QueryParam` con `@PathParam` (error común viniendo de `@RequestParam`/`@PathVariable`).
- No centralizar manejo de errores y devolver stack traces crudos al cliente.

## Checklist
- [ ] Sé mapear un CRUD REST completo con JAX-RS.
- [ ] Conozco la tabla de equivalencias con Spring MVC.
- [ ] Uso Bean Validation en los DTOs de entrada.
- [ ] Centralizo manejo de errores con `ExceptionMapper`.
- [ ] Entiendo cuándo usar `Uni`/`Multi` (Mutiny) vs estilo imperativo simple.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

