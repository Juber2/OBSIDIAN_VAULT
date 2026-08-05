---
tags:
  - Java
  - Patrones
  - Estructural
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - composite, árbol, jerarquía, tratamiento uniforme
---

## 🧠 Concepto

Compone objetos en **estructuras de árbol** y permite tratar objetos individuales y compuestos de forma uniforme.

**Cuándo usarlo:** Estructuras jerárquicas (carpetas/archivos, menús anidados, componentes de UI, organizaciones).

## Implementación

```java
// Componente — interfaz común para hojas y compuestos
public interface ComponenteUI {
    void render(String indentacion);
    int getAncho();
}

// Hoja — nodo sin hijos
public class Boton implements ComponenteUI {
    private final String label;
    private final int ancho;

    public Boton(String label, int ancho) {
        this.label = label;
        this.ancho = ancho;
    }

    public void render(String indentacion) {
        System.out.println(indentacion + "[Boton: " + label + "]");
    }

    public int getAncho() { return ancho; }
}

// Compuesto — nodo con hijos
public class Panel implements ComponenteUI {
    private final String nombre;
    private final List<ComponenteUI> hijos = new ArrayList<>();

    public Panel(String nombre) {
        this.nombre = nombre;
    }

    public void agregar(ComponenteUI componente) {
        hijos.add(componente);
    }

    public void remover(ComponenteUI componente) {
        hijos.remove(componente);
    }

    public void render(String indentacion) {
        System.out.println(indentacion + "[Panel: " + nombre + "]");
        for (ComponenteUI hijo : hijos) {
            hijo.render(indentacion + "  "); // más indentación para hijos
        }
    }

    public int getAncho() {
        return hijos.stream().mapToInt(ComponenteUI::getAncho).sum();
    }
}

// Uso
Panel formulario = new Panel("Formulario Principal");

Panel seccionDatos = new Panel("Datos Personales");
seccionDatos.agregar(new Boton("Nombre", 200));
seccionDatos.agregar(new Boton("Email", 200));

Panel seccionAcciones = new Panel("Acciones");
seccionAcciones.agregar(new Boton("Guardar", 100));
seccionAcciones.agregar(new Boton("Cancelar", 100));

formulario.agregar(seccionDatos);
formulario.agregar(seccionAcciones);

formulario.render(""); // imprime toda la jerarquía
```

## 📌 Notas Rápidas
