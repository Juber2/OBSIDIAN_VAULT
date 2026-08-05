---
tags:
  - Java
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - Spring Boot
Palabras clave:
  - @RestController, @GetMapping, @PostMapping, @PathVariable, @RequestBody, ResponseEntity
---

## 🧠 Anotaciones principales

```java
@RestController          // @Controller + @ResponseBody
@RequestMapping("/api/usuarios")  // prefijo de rutas
public class UsuarioController {

    @GetMapping                   // GET /api/usuarios
    @GetMapping("/{id}")          // GET /api/usuarios/1
    @PostMapping                  // POST /api/usuarios
    @PutMapping("/{id}")          // PUT /api/usuarios/1
    @PatchMapping("/{id}")        // PATCH /api/usuarios/1
    @DeleteMapping("/{id}")       // DELETE /api/usuarios/1
}
```

## Parámetros de entrada

```java
@RestController
@RequestMapping("/api/usuarios")
public class UsuarioController {

    // @PathVariable — variable en la URL
    @GetMapping("/{id}")
    public UsuarioResponse buscar(@PathVariable Long id) { ... }

    // @RequestParam — query param ?nombre=Juber&pagina=0
    @GetMapping
    public Page<UsuarioResponse> listar(
        @RequestParam(required = false) String nombre,
        @RequestParam(defaultValue = "0") int pagina,
        @RequestParam(defaultValue = "10") int tamaño
    ) { ... }

    // @RequestBody — cuerpo JSON
    @PostMapping
    public ResponseEntity<UsuarioResponse> crear(@Valid @RequestBody RegistroRequest request) { ... }

    // @RequestHeader — header HTTP
    @GetMapping("/perfil")
    public UsuarioResponse perfil(@RequestHeader("Authorization") String token) { ... }

    // @CookieValue
    @GetMapping("/session")
    public String sesion(@CookieValue("sessionId") String sessionId) { ... }
}
```

## ResponseEntity — control total del response

```java
@PostMapping
public ResponseEntity<UsuarioResponse> crear(@Valid @RequestBody RegistroRequest request) {
    UsuarioResponse response = service.crear(request);
    URI location = URI.create("/api/usuarios/" + response.getId());
    return ResponseEntity
        .created(location)   // 201 + Location header
        .body(response);
}

@GetMapping("/{id}")
public ResponseEntity<UsuarioResponse> buscar(@PathVariable Long id) {
    return service.buscarPorId(id)
        .map(ResponseEntity::ok)                   // 200
        .orElse(ResponseEntity.notFound().build()); // 404
}

@DeleteMapping("/{id}")
public ResponseEntity<Void> eliminar(@PathVariable Long id) {
    service.eliminar(id);
    return ResponseEntity.noContent().build(); // 204
}

// Con headers custom
return ResponseEntity.ok()
    .header("X-Total-Count", String.valueOf(total))
    .body(lista);
```

## HTTP Status codes estándar REST

| Operación | Status |
|-----------|--------|
| GET exitoso | 200 OK |
| POST exitoso (creado) | 201 Created |
| PUT/PATCH exitoso | 200 OK |
| DELETE exitoso | 204 No Content |
| No encontrado | 404 Not Found |
| Datos inválidos | 400 Bad Request |
| No autenticado | 401 Unauthorized |
| Sin permisos | 403 Forbidden |
| Error interno | 500 Internal Server Error |

## Ejemplo de controller completo

```java
@RestController
@RequestMapping("/api/v1/usuarios")
@RequiredArgsConstructor
public class UsuarioController {

    private final UsuarioService service;

    @GetMapping
    public ResponseEntity<Page<UsuarioResponse>> listar(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "nombre") String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return ResponseEntity.ok(service.listar(pageable));
    }

    @GetMapping("/{id}")
    public ResponseEntity<UsuarioResponse> buscar(@PathVariable Long id) {
        return ResponseEntity.ok(service.buscarPorId(id));
    }

    @PostMapping
    public ResponseEntity<UsuarioResponse> crear(@Valid @RequestBody CrearUsuarioRequest request) {
        UsuarioResponse response = service.crear(request);
        return ResponseEntity
            .created(URI.create("/api/v1/usuarios/" + response.getId()))
            .body(response);
    }

    @PutMapping("/{id}")
    public ResponseEntity<UsuarioResponse> actualizar(
            @PathVariable Long id,
            @Valid @RequestBody ActualizarUsuarioRequest request) {
        return ResponseEntity.ok(service.actualizar(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable Long id) {
        service.eliminar(id);
        return ResponseEntity.noContent().build();
    }
}
```

## 📌 Notas Rápidas
