---
tags:
  - Java
  - Spring
Creado: 2026-05-21
Relacionado:
  - Spring Core
  - Proxy
Palabras clave:
  - AOP, Aspect Oriented Programming, @Aspect, @Around, @Before, @After, pointcut, advice
---

## 🧠 Concepto

**AOP (Aspect Oriented Programming)** separa **cross-cutting concerns** (logging, transacciones, seguridad, auditoría) del código de negocio. Spring lo implementa con Dynamic Proxies.

## Terminología

| Término | Descripción |
|---------|-------------|
| **Aspect** | El módulo que encapsula el cross-cutting concern |
| **Advice** | La acción que ejecuta el aspecto (qué hacer) |
| **Pointcut** | La expresión que define dónde aplicar el aspecto (dónde) |
| **Join Point** | Un punto de ejecución donde se puede aplicar el aspecto |
| **Weaving** | Proceso de aplicar el aspecto al código objetivo |

## Dependencia

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## Aspecto de logging

```java
@Aspect
@Component
public class LoggingAspect {

    private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);

    // @Before — antes del método
    @Before("execution(* com.juber.miapp.service.*.*(..))")
    public void logAntes(JoinPoint joinPoint) {
        log.info("Llamando: {}.{}()",
            joinPoint.getTarget().getClass().getSimpleName(),
            joinPoint.getSignature().getName());
    }

    // @After — después, siempre (like finally)
    @After("execution(* com.juber.miapp.service.*.*(..))")
    public void logDespues(JoinPoint joinPoint) {
        log.info("Completado: {}", joinPoint.getSignature().getName());
    }

    // @AfterReturning — después, solo si no hay excepción
    @AfterReturning(pointcut = "execution(* com.juber.miapp.service.*.*(..))",
                    returning = "resultado")
    public void logRetorno(JoinPoint joinPoint, Object resultado) {
        log.info("Retornó: {}", resultado);
    }

    // @AfterThrowing — solo si lanzó excepción
    @AfterThrowing(pointcut = "execution(* com.juber.miapp.service.*.*(..))",
                   throwing = "ex")
    public void logExcepcion(JoinPoint joinPoint, Exception ex) {
        log.error("Error en {}: {}", joinPoint.getSignature().getName(), ex.getMessage());
    }

    // @Around — envuelve completamente el método (más poderoso)
    @Around("execution(* com.juber.miapp.service.*.*(..))")
    public Object medirTiempo(ProceedingJoinPoint joinPoint) throws Throwable {
        long inicio = System.currentTimeMillis();
        try {
            Object resultado = joinPoint.proceed(); // ejecutar el método original
            log.info("{} tardó {}ms", joinPoint.getSignature().getName(),
                System.currentTimeMillis() - inicio);
            return resultado;
        } catch (Exception e) {
            log.error("Error: {}", e.getMessage());
            throw e;
        }
    }
}
```

## Pointcut expressions

```java
// Cualquier método de cualquier clase en el paquete service
execution(* com.juber.miapp.service.*.*(..))

// Solo métodos públicos
execution(public * com.juber.miapp.service.*.*(..))

// Con tipo de retorno específico
execution(String com.juber.miapp.service.*.*(..))

// Cualquier clase anotada con @Service
@within(org.springframework.stereotype.Service)

// Cualquier método anotado con @Auditable
@annotation(com.juber.miapp.annotation.Auditable)
```

## Reutilizar pointcuts

```java
@Aspect
@Component
public class MiAspecto {

    @Pointcut("execution(* com.juber.miapp.service.*.*(..))")
    public void serviceLayer() {} // método vacío como referencia

    @Before("serviceLayer()")
    public void logAntes(JoinPoint jp) { ... }

    @Around("serviceLayer()")
    public Object medirTiempo(ProceedingJoinPoint pjp) throws Throwable { ... }
}
```

## Aspecto de auditoría con anotación custom

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    String accion();
}

@Aspect
@Component
public class AuditoriaAspect {
    @Autowired
    private AuditoriaRepository auditoriaRepo;

    @Around("@annotation(auditable)")
    public Object auditar(ProceedingJoinPoint pjp, Auditable auditable) throws Throwable {
        Object resultado = pjp.proceed();
        auditoriaRepo.save(new Auditoria(auditable.accion(), LocalDateTime.now()));
        return resultado;
    }
}

// Uso
@Auditable(accion = "CREAR_USUARIO")
public UsuarioResponse crear(RegistroRequest request) { ... }
```

## ⚠️ Errores Comunes

> [!warning] AOP solo funciona con beans de Spring
> Si `new MiServicio()` en lugar de `@Autowired`, el proxy no se aplica y el aspecto no funciona.

> [!warning] `@Transactional` en métodos privados no funciona
> Spring AOP usa proxies que no pueden interceptar llamadas internas del mismo objeto ni métodos privados.

## 📌 Notas Rápidas
- `@Transactional`, `@Cacheable`, `@Async` y `@CircuitBreaker` son todos AOP aplicado por Spring internamente — entender AOP explica por qué existe el "self-invocation problem" en todos ellos por igual.
- `@Around` es el advice más poderoso (controla si el método se ejecuta, puede modificar argumentos/resultado) pero también el más fácil de usar mal — para casos simples, `@Before`/`@AfterReturning` son más claros de leer.
- El weaving en Spring AOP ocurre en runtime vía proxies (a diferencia de AspectJ, que teje el bytecode en compile-time) — más simple pero con la limitación de que solo intercepta llamadas externas al bean, no internas.
- Una anotación custom + `@Around` con `@annotation(...)` como pointcut es el patrón estándar para funcionalidad transversal reutilizable (auditoría, rate limiting, métricas) sin ensuciar el código de negocio.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

