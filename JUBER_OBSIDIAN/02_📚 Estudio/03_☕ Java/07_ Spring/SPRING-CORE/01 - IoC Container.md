---
tags:
  - Java
  - Spring
Creado: 2026-05-21
Relacionado:
  - Spring Core
Palabras clave:
  - IoC, Inversion of Control, ApplicationContext, BeanFactory, contenedor Spring
---

## 🧠 Concepto

**IoC (Inversion of Control)** es el principio donde el control de la creación y ciclo de vida de los objetos se invierte: en lugar de que tu código cree sus dependencias, el **contenedor** las crea y las inyecta.

## Sin IoC vs Con IoC

```java
// Sin IoC — tu código controla la creación
public class PedidoService {
    private UsuarioRepository repo = new UsuarioRepository(); // tú lo creas
    private EmailService email = new EmailService();          // tú lo creas
}

// Con IoC — el contenedor lo controla
@Service
public class PedidoService {
    private final UsuarioRepository repo;   // Spring lo inyecta
    private final EmailService email;       // Spring lo inyecta

    public PedidoService(UsuarioRepository repo, EmailService email) {
        this.repo = repo;
        this.email = email;
    }
}
```

## ApplicationContext — el contenedor

```java
// Forma manual (sin Spring Boot)
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);

// Obtener bean
PedidoService servicio = ctx.getBean(PedidoService.class);

// Spring Boot lo hace automáticamente al arrancar
```

## Cómo Spring descubre los beans

```java
// 1. @ComponentScan — escanea el paquete y sus subpaquetes
@SpringBootApplication // incluye @ComponentScan del paquete raíz
public class MiApp { }

// 2. Anotaciones estereotipo
@Component       // bean genérico
@Service         // capa de servicio
@Repository      // capa de acceso a datos
@Controller      // controlador MVC
@RestController  // controlador REST

// 3. @Bean en clase @Configuration
@Configuration
public class AppConfig {
    @Bean
    public ConexionExterna conexion() {
        return new ConexionExterna("jdbc:postgresql://...");
    }
}
```

## Ciclo de vida de un Bean

```
Instanciación
      ↓
Inyección de dependencias
      ↓
@PostConstruct (inicialización personalizada)
      ↓
En uso por la aplicación
      ↓
@PreDestroy (limpieza antes de destruir)
      ↓
Destrucción
```

```java
@Component
public class ConexionDB {

    @PostConstruct
    public void inicializar() {
        System.out.println("Conectando a la BD...");
    }

    @PreDestroy
    public void cerrar() {
        System.out.println("Cerrando conexión...");
    }
}
```

## BeanFactory vs ApplicationContext

| | BeanFactory | ApplicationContext |
|--|-------------|-------------------|
| Lazy loading | Sí | No (eager por defecto) |
| Internacionalización | No | Sí |
| Eventos | No | Sí |
| AOP | Limitado | Completo |
| Uso | Aplicaciones muy ligeras | Aplicaciones estándar |

> [!tip] Siempre usar `ApplicationContext`. `BeanFactory` es raramente necesario.

## 📌 Notas Rápidas
- IoC es el principio general; DI (Dependency Injection) es la técnica más común para implementarlo — en la práctica se usan casi como sinónimos en Spring.
- `ApplicationContext` es eager por defecto (crea todos los singletons al arrancar) — esto hace que errores de configuración se detecten al inicio, no en medio de producción.
- Las anotaciones estereotipo (`@Service`, `@Repository`, `@Controller`) son semánticamente `@Component` con significado adicional — `@Repository` además traduce excepciones de persistencia a `DataAccessException` de Spring.
- `@PostConstruct`/`@PreDestroy` son estándar Jakarta, no específicos de Spring — funcionan igual en cualquier framework que los soporte.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

