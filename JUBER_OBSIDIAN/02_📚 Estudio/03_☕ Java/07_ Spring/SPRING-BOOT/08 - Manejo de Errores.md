---
tags:
  - Java
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - Spring Boot
  - REST Controllers
Palabras clave:
  - @RestControllerAdvice, @ExceptionHandler, ProblemDetail, RFC 7807, manejo de excepciones
---

## 🧠 Estrategia de manejo de errores

```
Capa Controller/Service → lanza excepciones de dominio
        ↓
@RestControllerAdvice → captura y transforma en respuesta HTTP
        ↓
Cliente recibe respuesta JSON estandarizada
```

## Excepciones de dominio

```java
// Excepción base
public abstract class AppException extends RuntimeException {
    public AppException(String message) { super(message); }
    public AppException(String message, Throwable cause) { super(message, cause); }
}

// Excepciones específicas
public class NotFoundException extends AppException {
    public NotFoundException(String recurso, Object id) {
        super(recurso + " con id " + id + " no encontrado");
    }
}

public class ConflictException extends AppException {
    public ConflictException(String message) { super(message); }
}

public class ForbiddenException extends AppException {
    public ForbiddenException(String message) { super(message); }
}
```

## @RestControllerAdvice global

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(NotFoundException ex, HttpServletRequest req) {
        log.warn("Not found: {}", ex.getMessage());
        return buildError("NOT_FOUND", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(ConflictException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleConflict(ConflictException ex, HttpServletRequest req) {
        return buildError("CONFLICT", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(ForbiddenException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResponse handleForbidden(ForbiddenException ex, HttpServletRequest req) {
        return buildError("FORBIDDEN", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex, HttpServletRequest req) {
        List<String> errores = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .sorted()
            .toList();
        return buildError("VALIDATION_ERROR", "Datos inválidos", req.getRequestURI(), errores);
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex, HttpServletRequest req) {
        log.error("Error interno", ex); // log completo en errores inesperados
        return buildError("SERVER_ERROR", "Error interno del servidor", req.getRequestURI());
    }

    private ErrorResponse buildError(String codigo, String mensaje, String path) {
        return buildError(codigo, mensaje, path, null);
    }

    private ErrorResponse buildError(String codigo, String mensaje, String path, List<String> detalles) {
        return ErrorResponse.builder()
            .codigo(codigo)
            .mensaje(mensaje)
            .path(path)
            .detalles(detalles)
            .timestamp(LocalDateTime.now())
            .build();
    }
}
```

## RFC 7807 — ProblemDetail (Spring 6 / Boot 3+)

```java
// Spring Boot 3 incluye soporte nativo para RFC 7807
@ExceptionHandler(NotFoundException.class)
public ProblemDetail handleNotFound(NotFoundException ex) {
    ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    pd.setTitle("Recurso no encontrado");
    pd.setType(URI.create("https://api.miapp.com/errores/not-found"));
    return pd;
}

// Respuesta:
// {
//   "type": "https://api.miapp.com/errores/not-found",
//   "title": "Recurso no encontrado",
//   "status": 404,
//   "detail": "Usuario con id 99 no encontrado"
// }
```

## Uso en el servicio

```java
@Service
public class UsuarioService {
    public UsuarioResponse buscarPorId(Long id) {
        return repository.findById(id)
            .map(mapper::toResponse)
            .orElseThrow(() -> new NotFoundException("Usuario", id));
    }

    public UsuarioResponse crear(CrearUsuarioRequest request) {
        if (repository.existsByEmail(request.email())) {
            throw new ConflictException("El email " + request.email() + " ya está en uso");
        }
        return mapper.toResponse(repository.save(mapper.toEntity(request)));
    }
}
```

## 📌 Notas Rápidas
- Un `@RestControllerAdvice` global evita repetir `try/catch` en cada controller — centraliza la traducción de excepciones de dominio a respuestas HTTP.
- El handler genérico de `Exception.class` debe loguear el stack trace completo (`log.error`) pero **nunca** devolverlo al cliente — filtrar detalles internos es un riesgo de seguridad.
- `ProblemDetail` (RFC 7807, nativo desde Spring Boot 3) es el estándar moderno de error response — preferible a un DTO de error propio si no hay requisitos específicos de formato.
- Las excepciones de dominio no deberían extender `Exception` (checked) sino `RuntimeException` — evita forzar `throws` en cada firma de método de servicio.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

