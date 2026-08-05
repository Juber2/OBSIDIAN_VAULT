---
tags:
  - Java
  - Fundamentos
  - POO
Creado: 2026-05-21
Relacionado:
  - Java Fundamentos
  - Herencia y Polimorfismo
Palabras clave:
  - interface, abstract, implements, contrato
---

## 🧠 Clase Abstracta

No se puede instanciar directamente. Define un molde parcial que las subclases deben completar.

```java
public abstract class Figura {
    protected String color;

    public Figura(String color) {
        this.color = color;
    }

    // Método abstracto — sin implementación, las subclases DEBEN implementarlo
    public abstract double calcularArea();

    // Método concreto — tiene implementación, se hereda
    public void mostrarColor() {
        System.out.println("Color: " + color);
    }
}

public class Circulo extends Figura {
    private double radio;

    public Circulo(String color, double radio) {
        super(color);
        this.radio = radio;
    }

    @Override
    public double calcularArea() {
        return Math.PI * radio * radio;
    }
}
```

## 🧠 Interface

Define un **contrato** — qué debe hacer una clase, no cómo. Se usa `implements`.

```java
public interface Volable {
    void volar(); // por defecto public abstract

    // Desde Java 8: métodos con implementación por defecto
    default void aterrizar() {
        System.out.println("Aterrizando...");
    }
}

public class Pajaro implements Volable {
    @Override
    public void volar() {
        System.out.println("El pájaro vuela");
    }
}
```

Una clase puede implementar **múltiples interfaces**:

```java
public class SuperHeroe implements Volable, Corrible, Nadable {
    // debe implementar todos los métodos abstractos de cada interface
}
```

## Comparativa

| Aspecto | Clase Abstracta | Interface |
|---------|-----------------|-----------|
| Instanciar | ❌ No | ❌ No |
| Herencia | `extends` (solo una) | `implements` (múltiples) |
| Constructor | ✅ Sí | ❌ No |
| Atributos de instancia | ✅ Sí | Solo `public static final` |
| Métodos concretos | ✅ Sí | Solo desde Java 8 (`default`) |
| Uso típico | Compartir código base | Definir contratos |

> [!tip] Cuándo usar cada uno
> - **Clase abstracta** → cuando las clases hijas comparten comportamiento común además del contrato
> - **Interface** → cuando solo necesitas definir un contrato, sin compartir implementación

## ⚠️ Errores Comunes

## 📌 Notas Rápidas
