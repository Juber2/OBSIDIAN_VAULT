---
tags:
  - Java
  - Patrones
  - Creacional
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - builder, construcción paso a paso, fluent API, Lombok Builder
---

## 🧠 Concepto

Construye objetos complejos **paso a paso**, separando la construcción de la representación. Evita constructores con muchos parámetros (telescoping constructor anti-pattern).

**Cuándo usarlo:** Objetos con muchos campos opcionales, construcción en múltiples pasos, objetos inmutables complejos.

## El problema (telescoping constructor)

```java
// ❌ Difícil de leer, propenso a errores de orden
new Pizza("grande", "fina", true, false, true, "mozzarella", "tomate");
```

## Implementación

```java
public class Pizza {
    // campos — algunos opcionales
    private final String tamaño;   // requerido
    private final String masa;     // requerido
    private final String queso;    // opcional
    private final boolean peperoni; // opcional
    private final boolean champiñones; // opcional

    private Pizza(Builder builder) {
        this.tamaño = builder.tamaño;
        this.masa = builder.masa;
        this.queso = builder.queso;
        this.peperoni = builder.peperoni;
        this.champiñones = builder.champiñones;
    }

    // Builder estático anidado
    public static class Builder {
        private final String tamaño; // requerido
        private final String masa;   // requerido
        private String queso = "mozzarella";
        private boolean peperoni = false;
        private boolean champiñones = false;

        public Builder(String tamaño, String masa) {
            this.tamaño = tamaño;
            this.masa = masa;
        }

        public Builder queso(String queso) {
            this.queso = queso;
            return this; // retorna this para encadenar
        }

        public Builder conPeperoni() {
            this.peperoni = true;
            return this;
        }

        public Builder conChampiñones() {
            this.champiñones = true;
            return this;
        }

        public Pizza build() {
            return new Pizza(this);
        }
    }
}

// Uso — legible y expresivo
Pizza pizza = new Pizza.Builder("grande", "fina")
    .queso("parmesano")
    .conPeperoni()
    .build();
```

## Con Lombok (uso real en proyectos)

```java
@Builder
@Getter
public class Usuario {
    private final Long id;
    private final String nombre;
    private final String email;
    private final String telefono; // opcional
    private final LocalDate fechaNacimiento; // opcional
}

// Uso
Usuario usuario = Usuario.builder()
    .id(1L)
    .nombre("Juber")
    .email("juber@email.com")
    .build();
```

## Builder con validaciones

```java
public Pizza build() {
    // Validar antes de construir
    Objects.requireNonNull(tamaño, "tamaño es requerido");
    if (tamaño.isEmpty()) throw new IllegalStateException("tamaño no puede estar vacío");
    return new Pizza(this);
}
```

## 📌 Notas Rápidas
