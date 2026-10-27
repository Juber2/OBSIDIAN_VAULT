---
tags:
  - Java
  - Patrones
  - Estructural
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - decorator, envolver, añadir comportamiento, composición sobre herencia
---

## 🧠 Concepto

Añade **comportamiento dinámicamente** a un objeto sin modificar su clase. Se hace envolviendo el objeto en un decorador que implementa la misma interfaz.

**Cuándo usarlo:** Añadir responsabilidades opcionales, evitar explosión de subclases, funcionalidad en capas (logging, caché, compresión).

## Implementación

```java
// Componente base
public interface Cafe {
    double getCosto();
    String getDescripcion();
}

// Componente concreto
public class CafeSimple implements Cafe {
    public double getCosto() { return 1.0; }
    public String getDescripcion() { return "Café simple"; }
}

// Decorador base (abstracto)
public abstract class CafeDecorator implements Cafe {
    protected final Cafe cafe;

    public CafeDecorator(Cafe cafe) {
        this.cafe = cafe;
    }
}

// Decoradores concretos
public class ConLeche extends CafeDecorator {
    public ConLeche(Cafe cafe) { super(cafe); }

    public double getCosto() { return cafe.getCosto() + 0.5; }
    public String getDescripcion() { return cafe.getDescripcion() + ", leche"; }
}

public class ConAzucar extends CafeDecorator {
    public ConAzucar(Cafe cafe) { super(cafe); }

    public double getCosto() { return cafe.getCosto() + 0.2; }
    public String getDescripcion() { return cafe.getDescripcion() + ", azúcar"; }
}

public class ConCaramelo extends CafeDecorator {
    public ConCaramelo(Cafe cafe) { super(cafe); }

    public double getCosto() { return cafe.getCosto() + 0.7; }
    public String getDescripcion() { return cafe.getDescripcion() + ", caramelo"; }
}

// Uso — se pueden combinar libremente
Cafe miCafe = new ConCaramelo(new ConAzucar(new ConLeche(new CafeSimple())));
System.out.println(miCafe.getDescripcion()); // "Café simple, leche, azúcar, caramelo"
System.out.println(miCafe.getCosto());        // 2.4
```

## Ejemplo real — Decorator para logging

```java
public interface ServicioUsuario {
    Usuario buscarPorId(Long id);
    void guardar(Usuario usuario);
}

public class ServicioUsuarioImpl implements ServicioUsuario {
    public Usuario buscarPorId(Long id) { ... }
    public void guardar(Usuario usuario) { ... }
}

// Decorador de logging
public class LoggingServicioUsuario implements ServicioUsuario {
    private final ServicioUsuario delegado;
    private final Logger log = LoggerFactory.getLogger(getClass());

    public LoggingServicioUsuario(ServicioUsuario delegado) {
        this.delegado = delegado;
    }

    public Usuario buscarPorId(Long id) {
        log.info("Buscando usuario con id: {}", id);
        long inicio = System.currentTimeMillis();
        Usuario resultado = delegado.buscarPorId(id);
        log.info("Búsqueda completada en {}ms", System.currentTimeMillis() - inicio);
        return resultado;
    }
}

// En Spring, AOP hace esto automáticamente con @Aspect
```

## Decorator vs Herencia

| Herencia | Decorator |
|----------|-----------|
| Estático (en compilación) | Dinámico (en runtime) |
| Explosión de subclases | Combinaciones libres |
| Acoplado a jerarquía | Composición flexible |

## 📌 Notas Rápidas
- Las clases `InputStream`/`OutputStream` del propio JDK (`BufferedInputStream`, `GZIPOutputStream`) son el ejemplo canónico de Decorator — se envuelven unas a otras para ir sumando comportamiento.
- En Spring, AOP (`@Aspect`, `@Transactional`, `@Cacheable`) logra el mismo efecto que un Decorator manual, pero generado por proxies dinámicos en vez de escribir clases wrapper a mano.
- "Composición sobre herencia" es la frase clave que resume por qué existe este patrón — evita una explosión combinatoria de subclases para cada combinación posible.
- El orden de envoltura importa: `new ConAzucar(new ConLeche(cafe))` no es lo mismo que `new ConLeche(new ConAzucar(cafe))` si el costo o la lógica dependen del orden.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

