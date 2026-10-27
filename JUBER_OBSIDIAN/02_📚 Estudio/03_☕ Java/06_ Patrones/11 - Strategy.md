---
tags:
  - Java
  - Patrones
  - Comportamiento
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - strategy, algoritmo intercambiable, comportamiento variable, lambdas
---

## 🧠 Concepto

Define una familia de algoritmos, los encapsula y los hace **intercambiables**. El cliente elige el algoritmo en runtime.

**Cuándo usarlo:** Múltiples variantes de un algoritmo, reemplazar condicionales grandes, comportamiento configurable.

## Implementación

```java
// Estrategia (interfaz)
public interface EstrategiaOrdenamiento {
    void ordenar(int[] array);
}

// Estrategias concretas
public class BubbleSort implements EstrategiaOrdenamiento {
    public void ordenar(int[] array) {
        System.out.println("Ordenando con BubbleSort...");
        // implementación
    }
}

public class QuickSort implements EstrategiaOrdenamiento {
    public void ordenar(int[] array) {
        System.out.println("Ordenando con QuickSort...");
        // implementación
    }
}

public class MergeSort implements EstrategiaOrdenamiento {
    public void ordenar(int[] array) {
        System.out.println("Ordenando con MergeSort...");
    }
}

// Contexto — usa la estrategia
public class Ordenador {
    private EstrategiaOrdenamiento estrategia;

    public Ordenador(EstrategiaOrdenamiento estrategia) {
        this.estrategia = estrategia;
    }

    public void setEstrategia(EstrategiaOrdenamiento estrategia) {
        this.estrategia = estrategia;
    }

    public void ordenar(int[] array) {
        estrategia.ordenar(array);
    }
}

// Uso
int[] datos = {5, 3, 8, 1, 9, 2};

Ordenador ordenador = new Ordenador(new QuickSort());
ordenador.ordenar(datos);

// Cambiar estrategia en runtime
ordenador.setEstrategia(new MergeSort());
ordenador.ordenar(datos);
```

## Con lambdas (más conciso)

```java
// EstrategiaOrdenamiento es una Functional Interface
Ordenador o1 = new Ordenador(arr -> Arrays.sort(arr)); // insertion sort JDK
Ordenador o2 = new Ordenador(arr -> {
    // custom algorithm
});
```

## Ejemplo real — estrategia de descuento

```java
@FunctionalInterface
public interface EstrategiaDescuento {
    double calcular(double precio);
}

public class CalculadoraPrecio {
    private EstrategiaDescuento estrategia;

    public CalculadoraPrecio(EstrategiaDescuento estrategia) {
        this.estrategia = estrategia;
    }

    public double calcularPrecioFinal(double precio) {
        return estrategia.calcular(precio);
    }
}

// Estrategias como constantes o lambdas
EstrategiaDescuento sinDescuento = precio -> precio;
EstrategiaDescuento descuento10 = precio -> precio * 0.90;
EstrategiaDescuento descuento20 = precio -> precio * 0.80;
EstrategiaDescuento descuentoNavidad = precio -> precio > 100 ? precio * 0.75 : precio * 0.85;

CalculadoraPrecio calc = new CalculadoraPrecio(descuentoNavidad);
calc.calcularPrecioFinal(150.0); // 112.5
```

## 📌 Notas Rápidas
- Con lambdas y functional interfaces, Strategy en Java moderno casi no necesita clases separadas — una `Map<String, Function<...>>` de estrategias suele bastar.
- Reemplaza directamente cadenas largas de `if/else` o `switch` que eligen comportamiento — si ves un `switch` que crece cada vez que se agrega un caso de negocio, es candidato a Strategy.
- Se combina naturalmente con Spring: inyectar `List<EstrategiaX>` y elegir la correcta en runtime por un identificador, en vez de un `if` gigante.
- Distinto de Template Method: Strategy cambia el **algoritmo completo** vía composición; Template Method fija la estructura y varía **pasos puntuales** vía herencia.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

