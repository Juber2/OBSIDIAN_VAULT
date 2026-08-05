---
tags:
  - Java
  - Fundamentos
  - POO
Creado: 2026-05-21
Relacionado:
  - Java Fundamentos
  - POO - Clases y Objetos
Palabras clave:
  - herencia, extends, polimorfismo, override, super
---

## 🧠 Herencia

Permite que una clase **herede** atributos y métodos de otra. Se usa `extends`.

```java
// Clase padre (superclase)
public class Animal {
    protected String nombre;

    public Animal(String nombre) {
        this.nombre = nombre;
    }

    public void hacerSonido() {
        System.out.println("...");
    }
}

// Clase hija (subclase)
public class Perro extends Animal {

    public Perro(String nombre) {
        super(nombre); // llama al constructor del padre
    }

    @Override
    public void hacerSonido() {
        System.out.println(nombre + " dice: ¡Guau!");
    }
}
```

## super

Referencia a la clase padre. Permite llamar a su constructor o métodos.

```java
super(nombre);           // constructor del padre
super.hacerSonido();     // método del padre
```

## @Override

Indica que estás sobreescribiendo un método del padre. No es obligatorio pero es buena práctica — el compilador verifica que el método exista en el padre.

## 🧠 Polimorfismo

Un objeto de tipo padre puede referenciar instancias de clases hijas.

```java
Animal animal1 = new Perro("Rex");
Animal animal2 = new Gato("Luna");

animal1.hacerSonido(); // Guau (ejecuta el método de Perro)
animal2.hacerSonido(); // Miau (ejecuta el método de Gato)
```

La decisión de qué método ejecutar se toma en **tiempo de ejecución** (dynamic dispatch).

## Reglas importantes

- Java solo permite herencia **simple** (una clase solo puede extender una clase)
- Para múltiple herencia se usan **interfaces** → [[07 - Interfaces y Abstracciones]]
- Una clase declarada `final` no puede ser extendida
- Un método declarado `final` no puede ser sobreescrito

## ⚠️ Errores Comunes

> [!warning] Olvidar `super()` en el constructor
> Si el padre no tiene constructor vacío, el hijo **debe** llamar a `super(...)` como primera línea del constructor.

## 📌 Notas Rápidas
