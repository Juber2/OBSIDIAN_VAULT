---
tags:
  - Java
  - Quarkus
Creado: 2026-08-09
Relacionado:
  - Quarkus
---

# CDI e Inyección de Dependencias

Quarkus usa **CDI (Contexts and Dependency Injection)**, el estándar de Jakarta EE, en vez del contenedor propio de Spring. Los conceptos son similares (inyección por constructor/campo, scopes, interfaces), pero las anotaciones y el detrás de escena cambian.

## Beans básicos

```java
@ApplicationScoped
public class SaludoService {
    public String saludar(String nombre) {
        return "Hola, " + nombre;
    }
}
```

```java
@Path("/saludo")
public class SaludoResource {

    @Inject
    SaludoService saludoService;

    @GET
    @Path("/{nombre}")
    public String saludar(@PathParam("nombre") String nombre) {
        return saludoService.saludar(nombre);
    }
}
```

Inyección por constructor (preferida, igual que en Spring):
```java
@ApplicationScoped
public class PedidoService {
    private final SaludoService saludoService;

    @Inject
    public PedidoService(SaludoService saludoService) {
        this.saludoService = saludoService;
    }
}
```

## Scopes de CDI

| Scope | Equivalente Spring | Ciclo de vida |
|---|---|---|
| `@ApplicationScoped` | `@Singleton`/`@Component` (default) | una instancia por toda la app |
| `@RequestScoped` | `@RequestScope` | una instancia por request HTTP |
| `@Singleton` (CDI) | — | como ApplicationScoped pero sin proxy (sin soporte de contexto real) |
| `@Dependent` | `@Prototype` | nueva instancia cada vez que se inyecta |

`@ApplicationScoped` es el default recomendado para la mayoría de servicios — equivalente conceptual al singleton de Spring.

## Interfaces y múltiples implementaciones

```java
public interface NotificacionService {
    void enviar(String mensaje);
}

@ApplicationScoped
public class EmailNotificacion implements NotificacionService {
    public void enviar(String mensaje) { /* ... */ }
}

@ApplicationScoped
@Alternative
@Priority(1)
public class SmsNotificacion implements NotificacionService {
    public void enviar(String mensaje) { /* ... */ }
}
```

`@Alternative` + `@Priority` resuelve ambigüedad cuando hay más de una implementación — análogo a `@Primary`/`@Qualifier` en Spring.

## Producers: crear beans desde código (no desde clase anotada)

```java
@ApplicationScoped
public class ConfigProducer {

    @Produces
    @ApplicationScoped
    public ObjectMapper objectMapper() {
        return new ObjectMapper().findAndRegisterModules();
    }
}
```

Equivalente a un `@Bean` dentro de una `@Configuration` en Spring.

## Eventos (CDI events) — desacoplar sin llamada directa

```java
@ApplicationScoped
public class PedidoService {
    @Inject
    Event<PedidoCreadoEvent> pedidoCreadoEvent;

    public void crear(Pedido pedido) {
        // ...
        pedidoCreadoEvent.fire(new PedidoCreadoEvent(pedido.getId()));
    }
}

@ApplicationScoped
public class NotificadorPedido {
    public void onPedidoCreado(@Observes PedidoCreadoEvent evento) {
        // enviar email, etc.
    }
}
```

## Errores comunes

- Usar `@RequestScoped` en beans que deberían ser `@ApplicationScoped`, generando overhead innecesario.
- Olvidar `@Alternative`/`@Priority` cuando hay múltiples implementaciones de una interfaz → ambigüedad en build time (Quarkus falla temprano, a diferencia de fallar en runtime como a veces pasa en Spring).
- Suponer que CDI y Spring DI son 100% intercambiables conceptualmente — el modelo de proxies/interceptors difiere y afecta cómo se comportan con compilación nativa.

## Checklist
- [ ] Sé declarar beans con `@ApplicationScoped` y otros scopes de CDI.
- [ ] Entiendo la diferencia práctica entre los scopes de CDI y los de Spring.
- [ ] Sé resolver ambigüedad de múltiples implementaciones con `@Alternative`/`@Priority`.
- [ ] Sé usar `@Produces` para beans que no vienen de una clase anotada directamente.
- [ ] Entiendo CDI events como forma de desacoplar componentes.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

