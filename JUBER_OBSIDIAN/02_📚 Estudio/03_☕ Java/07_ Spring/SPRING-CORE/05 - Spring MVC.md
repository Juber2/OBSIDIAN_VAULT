---
tags:
  - Java
  - Spring
Creado: 2026-05-21
Relacionado:
  - Spring Core
  - Spring Boot
Palabras clave:
  - Spring MVC, DispatcherServlet, HandlerMapping, ViewResolver, ModelAndView
---

## 🧠 Concepto

**Spring MVC** es el framework web de Spring, basado en el patrón MVC. El componente central es el `DispatcherServlet` que actúa como front controller.

## Flujo de una petición

```
HTTP Request
     ↓
DispatcherServlet
     ↓
HandlerMapping → encuentra el Controller correcto
     ↓
HandlerAdapter → invoca el método del Controller
     ↓
Controller → procesa la petición → devuelve ModelAndView o ResponseBody
     ↓
ViewResolver → (si no es REST) resuelve la vista
     ↓
HTTP Response
```

## Componentes principales

```java
// @Controller — devuelve vistas (Thymeleaf, JSP)
@Controller
public class VistaController {
    @GetMapping("/inicio")
    public String inicio(Model model) {
        model.addAttribute("mensaje", "Bienvenido");
        return "inicio"; // nombre de la vista
    }
}

// @RestController — devuelve JSON/XML (@Controller + @ResponseBody)
@RestController
@RequestMapping("/api/usuarios")
public class UsuarioController {
    @GetMapping("/{id}")
    public UsuarioResponse buscar(@PathVariable Long id) { ... }
}
```

## HandlerInterceptor — interceptar peticiones

```java
@Component
public class LoggingInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                              HttpServletResponse response,
                              Object handler) {
        log.info("Request: {} {}", request.getMethod(), request.getRequestURI());
        return true; // continuar la cadena
    }

    @Override
    public void postHandle(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler,
                            ModelAndView modelAndView) {
        // después del handler, antes de render
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Object handler, Exception ex) {
        // después de todo
    }
}

// Registrar el interceptor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    private LoggingInterceptor loggingInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loggingInterceptor)
            .addPathPatterns("/api/**")
            .excludePathPatterns("/api/auth/**");
    }
}
```

## @ControllerAdvice — manejo global de errores

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(NotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        List<String> errores = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .toList();
        return new ErrorResponse("VALIDATION_ERROR", errores.toString());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        return new ErrorResponse("SERVER_ERROR", "Error interno del servidor");
    }
}
```

## ContentNegotiation — JSON vs XML

```java
// Spring Boot configura automáticamente Jackson para JSON
// Para XML agregar:
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>

// El cliente decide el formato con el header Accept:
// Accept: application/json  → JSON
// Accept: application/xml   → XML
```

## 📌 Notas Rápidas
