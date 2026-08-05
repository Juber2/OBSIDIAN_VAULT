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
  - clases, objetos, constructor, encapsulamiento, getters, setters
---

## 🧠 Concepto

Una **clase** es un molde. Un **objeto** es una instancia de ese molde.

## Estructura de una clase

```java
public class Persona {

    // Atributos (estado)
    private String nombre;
    private int edad;

    // Constructor
    public Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }

    // Constructor vacío
    public Persona() {}

    // Getters y Setters (encapsulamiento)
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }

    public int getEdad() { return edad; }
    public void setEdad(int edad) { this.edad = edad; }

    // Métodos (comportamiento)
    public void presentarse() {
        System.out.println("Hola, soy " + nombre + " y tengo " + edad + " años.");
    }

    // toString (buena práctica)
    @Override
    public String toString() {
        return "Persona{nombre='" + nombre + "', edad=" + edad + "}";
    }
}
```

## Crear objetos (instanciar)

```java
Persona persona1 = new Persona("Juber", 25);
persona1.presentarse();

Persona persona2 = new Persona();
persona2.setNombre("Ana");
```

## this

Hace referencia a la instancia actual de la clase. Se usa para diferenciar atributos de parámetros con el mismo nombre.

```java
public void setNombre(String nombre) {
    this.nombre = nombre; // this.nombre = atributo, nombre = parámetro
}
```

## Los 4 pilares de la POO

| Pilar | Descripción |
|-------|-------------|
| **Encapsulamiento** | Ocultar el estado interno, exponer solo lo necesario (private + getters/setters) |
| **Herencia** | Una clase puede heredar de otra (`extends`) → ver [[06 - Herencia y Polimorfismo]] |
| **Polimorfismo** | Un objeto puede comportarse de distintas formas → ver [[06 - Herencia y Polimorfismo]] |
| **Abstracción** | Ocultar complejidad, mostrar solo lo esencial → ver [[07 - Interfaces y Abstracciones]] |

## ⚠️ Errores Comunes

> [!warning] NullPointerException
> Si creas una variable de referencia pero no la inicializas, al usarla obtienes `NullPointerException`.
> ```java
> Persona p; // declarada pero no inicializada
> p.getNombre(); // ❌ NullPointerException
> ```

## 📌 Notas Rápidas
