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

> [!warning] Conflicto de métodos `default` al implementar dos interfaces
> Si dos interfaces tienen un método `default` con la misma firma, la clase debe sobrescribirlo explícitamente para resolver la ambigüedad.
> ```java
> interface A { default void hola() { System.out.println("A"); } }
> interface B { default void hola() { System.out.println("B"); } }
> class C implements A, B {
>     @Override
>     public void hola() { A.super.hola(); } // hay que elegir explícito
> }
> ```

> [!warning] Intentar instanciar directamente
> ```java
> Figura f = new Figura("rojo"); // ❌ no se puede instanciar una clase abstracta
> Volable v = new Volable(); // ❌ tampoco una interface
> ```

## 📌 Notas Rápidas
- Una interface puede tener campos, pero implícitamente son `public static final` (constantes).
- Desde Java 9, las interfaces también pueden tener métodos `private` (para reutilizar lógica entre métodos `default`).
- Preferí interfaces sobre clases abstractas cuando no necesitás compartir estado/implementación — es más flexible (una clase puede implementar varias).
- Las clases abstractas sí pueden tener constructor, aunque no se instancien directo — se ejecuta cuando una subclase concreta se instancia.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

