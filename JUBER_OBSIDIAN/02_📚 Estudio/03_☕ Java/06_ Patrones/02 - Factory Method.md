---
tags:
  - Java
  - Patrones
  - Creacional
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
  - Abstract Factory
Palabras clave:
  - factory method, creación de objetos, polimorfismo, desacoplamiento
---

## 🧠 Concepto

Define una interfaz para crear objetos, pero deja que las **subclases decidan** qué clase instanciar. El creador no conoce el tipo concreto.

**Cuándo usarlo:** Cuando la lógica de creación de objetos debe estar en subclases, o cuando no sabes de antemano qué tipo de objeto crear.

## Implementación

```java
// Producto (interfaz)
public interface Notificador {
    void enviar(String mensaje);
}

// Productos concretos
public class EmailNotificador implements Notificador {
    @Override
    public void enviar(String mensaje) {
        System.out.println("Email: " + mensaje);
    }
}

public class SmsNotificador implements Notificador {
    @Override
    public void enviar(String mensaje) {
        System.out.println("SMS: " + mensaje);
    }
}

// Creador abstracto
public abstract class NotificadorFactory {
    // Factory Method — cada subclase decide qué crear
    public abstract Notificador crearNotificador();

    // Lógica común que usa el factory method
    public void notificar(String mensaje) {
        Notificador notificador = crearNotificador();
        notificador.enviar(mensaje);
    }
}

// Creadores concretos
public class EmailFactory extends NotificadorFactory {
    @Override
    public Notificador crearNotificador() {
        return new EmailNotificador();
    }
}

public class SmsFactory extends NotificadorFactory {
    @Override
    public Notificador crearNotificador() {
        return new SmsNotificador();
    }
}

// Uso
NotificadorFactory factory = new EmailFactory();
factory.notificar("Hola mundo");
```

## Variante simple — Static Factory Method

```java
public class NotificadorFactory {
    public static Notificador crear(String tipo) {
        return switch (tipo.toLowerCase()) {
            case "email" -> new EmailNotificador();
            case "sms"   -> new SmsNotificador();
            case "push"  -> new PushNotificador();
            default -> throw new IllegalArgumentException("Tipo desconocido: " + tipo);
        };
    }
}

// Uso
Notificador n = NotificadorFactory.crear("email");
n.enviar("Hola");
```

## Factory en Spring

```java
@Component
public class NotificadorFactory {
    private final Map<String, Notificador> notificadores;

    public NotificadorFactory(List<Notificador> lista) {
        // Spring inyecta todos los beans que implementan Notificador
        this.notificadores = lista.stream()
            .collect(Collectors.toMap(
                n -> n.getClass().getSimpleName().toLowerCase(),
                Function.identity()
            ));
    }

    public Notificador obtener(String tipo) {
        return Optional.ofNullable(notificadores.get(tipo))
            .orElseThrow(() -> new IllegalArgumentException("Tipo: " + tipo));
    }
}
```

## 📌 Notas Rápidas
- La variante "Static Factory Method" es la más usada en código real Java (`List.of()`, `Optional.of()`) — más simple que la jerarquía completa con subclases.
- En Spring, inyectar `List<Interfaz>` para que el framework junte automáticamente todos los beans que implementan una interfaz es el equivalente idiomático a una factory.
- Factory Method resuelve "qué instanciar" sin que el código cliente conozca las clases concretas — clave para cumplir el principio de inversión de dependencias.
- No confundir con Abstract Factory: acá se crea **un** tipo de producto, no una familia completa.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

