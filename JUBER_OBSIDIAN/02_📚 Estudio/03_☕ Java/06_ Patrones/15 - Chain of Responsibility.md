---
tags:
  - Java
  - Patrones
  - Comportamiento
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - chain of responsibility, cadena, filtros, pipeline, middleware, handlers
---

## 🧠 Concepto

Pasa una petición a través de una **cadena de manejadores**. Cada manejador decide si la procesa o la pasa al siguiente.

**Cuándo usarlo:** Validaciones encadenadas, filtros HTTP, pipelines de procesamiento, middleware, sistemas de aprobación.

## Implementación

```java
// Handler (manejador)
public abstract class Aprobador {
    protected Aprobador siguiente;

    public Aprobador setNext(Aprobador siguiente) {
        this.siguiente = siguiente;
        return siguiente; // permite encadenar fluently
    }

    public abstract void procesar(Solicitud solicitud);
}

public class Solicitud {
    private final double monto;
    private final String tipo;

    public Solicitud(double monto, String tipo) {
        this.monto = monto;
        this.tipo = tipo;
    }

    public double getMonto() { return monto; }
    public String getTipo() { return tipo; }
}

// Manejadores concretos
public class AprobadorJunior extends Aprobador {
    public void procesar(Solicitud solicitud) {
        if (solicitud.getMonto() <= 1000) {
            System.out.println("Junior aprueba solicitud de $" + solicitud.getMonto());
        } else if (siguiente != null) {
            siguiente.procesar(solicitud);
        }
    }
}

public class AprobadorSenior extends Aprobador {
    public void procesar(Solicitud solicitud) {
        if (solicitud.getMonto() <= 10000) {
            System.out.println("Senior aprueba solicitud de $" + solicitud.getMonto());
        } else if (siguiente != null) {
            siguiente.procesar(solicitud);
        }
    }
}

public class Director extends Aprobador {
    public void procesar(Solicitud solicitud) {
        System.out.println("Director aprueba solicitud de $" + solicitud.getMonto());
    }
}

// Uso — construir la cadena
Aprobador junior = new AprobadorJunior();
Aprobador senior = new AprobadorSenior();
Aprobador director = new Director();

junior.setNext(senior).setNext(director); // encadenado fluent

junior.procesar(new Solicitud(500, "compra"));     // Junior aprueba
junior.procesar(new Solicitud(5000, "viaje"));     // Senior aprueba
junior.procesar(new Solicitud(50000, "proyecto")); // Director aprueba
```

## Implementación funcional con lista

```java
public class Pipeline<T> {
    private final List<Function<T, T>> pasos = new ArrayList<>();

    public Pipeline<T> agregar(Function<T, T> paso) {
        pasos.add(paso);
        return this;
    }

    public T ejecutar(T entrada) {
        T resultado = entrada;
        for (Function<T, T> paso : pasos) {
            resultado = paso.apply(resultado);
        }
        return resultado;
    }
}

// Uso
String resultado = new Pipeline<String>()
    .agregar(String::trim)
    .agregar(String::toLowerCase)
    .agregar(s -> s.replace(" ", "_"))
    .ejecutar("  Hola Mundo  ");
// "hola_mundo"
```

## Ejemplo real — filtros de validación

```java
@FunctionalInterface
public interface Filtro<T> {
    boolean pasar(T objeto);
}

public class ValidadorPedido {
    private final List<Filtro<Pedido>> filtros = new ArrayList<>();

    public ValidadorPedido agregar(Filtro<Pedido> filtro) {
        filtros.add(filtro);
        return this;
    }

    public boolean validar(Pedido pedido) {
        return filtros.stream().allMatch(f -> f.pasar(pedido));
    }
}

// Uso
ValidadorPedido validador = new ValidadorPedido()
    .agregar(p -> p.getMonto() > 0)
    .agregar(p -> p.getCliente() != null)
    .agregar(p -> p.getItems() != null && !p.getItems().isEmpty());

validador.validar(pedido); // true si pasa todos los filtros
```

## Relación con Spring

Los filtros de Spring Security y los `HandlerInterceptor` son implementaciones de Chain of Responsibility.

## 📌 Notas Rápidas
