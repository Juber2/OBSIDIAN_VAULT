---
tags:
  - Java
  - Spring
Creado: 2026-05-21
Relacionado:
  - Spring Core
  - IoC Container
Palabras clave:
  - @Autowired, @Qualifier, inyección constructor, inyección setter, inyección campo
---

## 🧠 Tipos de Inyección

### 1. Inyección por constructor (RECOMENDADA)

```java
@Service
public class PedidoService {
    private final UsuarioRepository usuarioRepo;
    private final ProductoRepository productoRepo;

    // Spring inyecta automáticamente (no necesita @Autowired desde Spring 4.3
    // si solo hay un constructor)
    public PedidoService(UsuarioRepository usuarioRepo,
                         ProductoRepository productoRepo) {
        this.usuarioRepo = usuarioRepo;
        this.productoRepo = productoRepo;
    }
}
```

**Ventajas:**
- Campos `final` → inmutabilidad garantizada
- Dependencias explícitas y obligatorias
- Fácil de testear (pasar mocks al constructor)
- Detecta dependencias circulares en startup

### 2. Inyección por setter

```java
@Service
public class ReporteService {
    private EmailService emailService;

    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

**Cuándo usar:** Solo para dependencias opcionales (`@Autowired(required = false)`).

### 3. Inyección por campo (NO RECOMENDADA en producción)

```java
@Service
public class UsuarioService {
    @Autowired // inyecta el campo directamente
    private UsuarioRepository repository;
}
```

**Problemas:**
- No puede ser `final`
- Difícil de testear sin el contexto de Spring
- Oculta las dependencias

## @Qualifier — cuando hay múltiples implementaciones

```java
public interface Notificador {
    void notificar(String mensaje);
}

@Component("email")
public class EmailNotificador implements Notificador { ... }

@Component("sms")
public class SmsNotificador implements Notificador { ... }

// Inyectar uno específico
@Service
public class AlertaService {
    private final Notificador notificador;

    public AlertaService(@Qualifier("email") Notificador notificador) {
        this.notificador = notificador;
    }
}
```

## @Primary — implementación por defecto

```java
@Component
@Primary // se usa cuando no se especifica @Qualifier
public class EmailNotificador implements Notificador { ... }
```

## @Value — inyectar valores de configuración

```java
@Service
public class ConfigService {
    @Value("${app.nombre}")
    private String appNombre;

    @Value("${app.max-intentos:3}") // 3 es el valor por defecto
    private int maxIntentos;

    @Value("${app.features.activo:false}")
    private boolean featureActivo;
}
```

## Dependencias circulares

```
A necesita B, B necesita A → Spring no puede resolverlo con inyección por constructor
```

```java
// ❌ Circular con constructor — lanza BeanCurrentlyInCreationException
// ✅ Solución: revisar el diseño (generalmente señal de mal diseño)
// ✅ O usar @Lazy en uno de los dos
@Service
public class ServiceA {
    private final ServiceB serviceB;
    public ServiceA(@Lazy ServiceB serviceB) { this.serviceB = serviceB; }
}
```

## 📌 Notas Rápidas
- Constructor injection es la única forma que permite campos `final` y detecta dependencias circulares en el arranque — por eso es la recomendada, no field injection.
- Con Lombok, `@RequiredArgsConstructor` genera el constructor automáticamente sobre campos `final` — combinación estándar en Spring moderno (ver [[06 - Lombok]]).
- Una dependencia circular casi siempre es señal de mal diseño (dos servicios que se necesitan mutuamente) — `@Lazy` es un parche, no una solución; lo correcto suele ser extraer la lógica compartida a un tercer servicio.
- `@Qualifier` + múltiples `@Component` con nombre es el equivalente Spring de resolver ambigüedad, similar a `@Primary`/`@Alternative` en CDI (Quarkus) — ver [[03 - CDI e Inyeccion de Dependencias]].

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

