---
tags:
  - Java
  - Patrones
  - Creacional
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
  - Factory Method
Palabras clave:
  - abstract factory, familia de objetos, productos relacionados
---

## 🧠 Concepto

Crea **familias de objetos relacionados** sin especificar sus clases concretas. Garantiza que los productos de una familia sean compatibles entre sí.

**Cuándo usarlo:** Cuando necesitas crear múltiples objetos que deben trabajar juntos y quieres poder cambiar toda la familia de una vez.

## Implementación

```java
// Productos abstractos
public interface Boton {
    void click();
}

public interface Checkbox {
    void seleccionar();
}

// Productos concretos — familia Windows
public class WindowsBoton implements Boton {
    public void click() { System.out.println("Windows Boton click"); }
}

public class WindowsCheckbox implements Checkbox {
    public void seleccionar() { System.out.println("Windows Checkbox"); }
}

// Productos concretos — familia MacOS
public class MacBoton implements Boton {
    public void click() { System.out.println("Mac Boton click"); }
}

public class MacCheckbox implements Checkbox {
    public void seleccionar() { System.out.println("Mac Checkbox"); }
}

// Abstract Factory
public interface UIFactory {
    Boton crearBoton();
    Checkbox crearCheckbox();
}

// Fábricas concretas
public class WindowsFactory implements UIFactory {
    public Boton crearBoton() { return new WindowsBoton(); }
    public Checkbox crearCheckbox() { return new WindowsCheckbox(); }
}

public class MacFactory implements UIFactory {
    public Boton crearBoton() { return new MacBoton(); }
    public Checkbox crearCheckbox() { return new MacCheckbox(); }
}

// Cliente — no conoce las clases concretas
public class Aplicacion {
    private final Boton boton;
    private final Checkbox checkbox;

    public Aplicacion(UIFactory factory) {
        this.boton = factory.crearBoton();
        this.checkbox = factory.crearCheckbox();
    }

    public void render() {
        boton.click();
        checkbox.seleccionar();
    }
}

// Uso
UIFactory factory = System.getProperty("os.name").contains("Mac")
    ? new MacFactory()
    : new WindowsFactory();

Aplicacion app = new Aplicacion(factory);
app.render();
```

## Diferencia con Factory Method

| | Factory Method | Abstract Factory |
|--|---------------|-----------------|
| Qué crea | Un producto | Familia de productos |
| Cómo | Subclases sobreescriben un método | Objeto independiente con múltiples métodos |
| Uso típico | Un tipo de objeto variable | Varios objetos que deben ser compatibles |

## 📌 Notas Rápidas
