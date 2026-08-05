---
tags:
  - Java
  - Spring
Creado: 2026-05-21
Relacionado:
  - Spring Core
Palabras clave:
  - singleton, prototype, request, session, @Scope, @Bean, @Configuration
---

## 🧠 Scopes de Beans

### Singleton (default)
Una sola instancia para todo el contexto de Spring.

```java
@Service  // singleton por defecto
public class UsuarioService { }

// Explícito
@Component
@Scope("singleton")
public class MiServicio { }
```

### Prototype
Nueva instancia cada vez que se solicita el bean.

```java
@Component
@Scope("prototype")
public class TareaTemporal {
    private String estado;
    // cada inyección o getBean() crea una instancia nueva
}
```

> [!warning] Prototype en Singleton
> Si un bean Singleton tiene un campo Prototype, el campo se inyecta una sola vez (en el startup). Para obtener siempre una instancia nueva, usar `ObjectFactory<T>` o `ApplicationContext.getBean()`.

```java
@Service
public class ServicioConPrototype {
    @Autowired
    private ObjectFactory<TareaTemporal> factory;

    public void ejecutar() {
        TareaTemporal tarea = factory.getObject(); // siempre nueva instancia
    }
}
```

### Request (solo aplicaciones web)
Una instancia por petición HTTP.

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ContextoPeticion {
    private String idUsuario;
}
```

### Session (solo aplicaciones web)
Una instancia por sesión HTTP.

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class CarritoCompras {
    private List<Producto> items = new ArrayList<>();
}
```

## @Configuration y @Bean

```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        RestTemplate rt = new RestTemplate();
        rt.setInterceptors(List.of(new LoggingInterceptor()));
        return rt;
    }

    @Bean
    public ObjectMapper objectMapper() {
        return JsonMapper.builder()
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
            .build();
    }

    // Bean con dependencia de otro bean
    @Bean
    public ApiClient apiClient(RestTemplate restTemplate) {
        return new ApiClient(restTemplate, "https://api.externa.com");
    }
}
```

## Condicionales

```java
// Bean solo si existe una propiedad
@Bean
@ConditionalOnProperty(name = "feature.cache.enabled", havingValue = "true")
public CacheManager cacheManager() {
    return new RedisCacheManager(...);
}

// Bean solo si no existe otro bean del mismo tipo
@Bean
@ConditionalOnMissingBean(EmailService.class)
public EmailService emailServiceDefecto() {
    return new MockEmailService();
}

// Bean solo en cierto perfil
@Bean
@Profile("dev")
public DataSource dataSourceDev() { ... }

@Bean
@Profile("prod")
public DataSource dataSourceProd() { ... }
```

## 📌 Notas Rápidas
