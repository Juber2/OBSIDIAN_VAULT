---
tags:
  - Java
  - Patrones
  - Comportamiento
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - observer, eventos, publisher, subscriber, event listener, desacoplamiento
---

## 🧠 Concepto

Define una relación **uno-a-muchos**: cuando un objeto cambia, todos sus dependientes son notificados automáticamente.

**Cuándo usarlo:** Eventos de UI, notificaciones, sistemas reactivos, mensajería desacoplada.

## Implementación

```java
// Observer (suscriptor)
public interface Observer {
    void actualizar(String evento, Object data);
}

// Subject (publicador)
public class SistemaPedidos {
    private final List<Observer> observers = new ArrayList<>();

    public void suscribir(Observer observer) {
        observers.add(observer);
    }

    public void desuscribir(Observer observer) {
        observers.remove(observer);
    }

    private void notificar(String evento, Object data) {
        observers.forEach(o -> o.actualizar(evento, data));
    }

    // Lógica de negocio
    public void crearPedido(Pedido pedido) {
        // ... lógica de creación
        notificar("PEDIDO_CREADO", pedido);
    }

    public void cancelarPedido(Long id) {
        // ... lógica de cancelación
        notificar("PEDIDO_CANCELADO", id);
    }
}

// Observers concretos
public class NotificadorEmail implements Observer {
    public void actualizar(String evento, Object data) {
        if ("PEDIDO_CREADO".equals(evento)) {
            System.out.println("Email enviado para pedido: " + data);
        }
    }
}

public class SistemaInventario implements Observer {
    public void actualizar(String evento, Object data) {
        if ("PEDIDO_CREADO".equals(evento)) {
            System.out.println("Inventario actualizado para: " + data);
        }
    }
}

// Uso
SistemaPedidos sistema = new SistemaPedidos();
sistema.suscribir(new NotificadorEmail());
sistema.suscribir(new SistemaInventario());

sistema.crearPedido(new Pedido(...)); // notifica a ambos
```

## Observer con lambdas

```java
public class EventBus {
    private final Map<String, List<Consumer<Object>>> handlers = new HashMap<>();

    public void subscribe(String evento, Consumer<Object> handler) {
        handlers.computeIfAbsent(evento, k -> new ArrayList<>()).add(handler);
    }

    public void publish(String evento, Object data) {
        handlers.getOrDefault(evento, Collections.emptyList())
                .forEach(h -> h.accept(data));
    }
}

// Uso
EventBus bus = new EventBus();
bus.subscribe("PEDIDO_CREADO", data -> System.out.println("Email: " + data));
bus.subscribe("PEDIDO_CREADO", data -> System.out.println("Stock: " + data));
bus.publish("PEDIDO_CREADO", pedido);
```

## Observer en Spring

```java
// Spring Events — observer desacoplado
public class PedidoCreadoEvent extends ApplicationEvent {
    private final Pedido pedido;
    public PedidoCreadoEvent(Object source, Pedido pedido) {
        super(source);
        this.pedido = pedido;
    }
}

@Service
public class PedidoService {
    @Autowired
    private ApplicationEventPublisher publisher;

    public void crearPedido(Pedido pedido) {
        // ...
        publisher.publishEvent(new PedidoCreadoEvent(this, pedido));
    }
}

@Component
public class EmailListener {
    @EventListener
    public void onPedidoCreado(PedidoCreadoEvent event) {
        System.out.println("Enviando email para: " + event.getPedido());
    }
}
```

## 📌 Notas Rápidas
- Spring Application Events (`ApplicationEventPublisher` + `@EventListener`) es Observer nativo del framework — evita escribir el mecanismo de suscripción a mano.
- Es la base conceptual de Event-Driven Architecture y de mensajería (Kafka/RabbitMQ) — la diferencia es que ahí el "Subject" y los "Observers" viven en procesos/servicios distintos, no en el mismo objeto en memoria.
- Cuidado con el orden de notificación si los observers tienen dependencias entre sí — el patrón no garantiza orden salvo que lo implementes explícitamente.
- Un observer que lanza una excepción puede frenar la notificación al resto — en Spring, `@EventListener` con `@Async` desacopla eso (a costa de perder el manejo síncrono de errores).

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

