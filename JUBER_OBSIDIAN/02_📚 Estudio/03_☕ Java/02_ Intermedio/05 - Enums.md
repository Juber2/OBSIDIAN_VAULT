---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
Palabras clave:
  - enum, enumeración, constantes, switch, atributos, métodos
---

## 🧠 Concepto

Un `enum` es un tipo especial que representa un conjunto fijo de constantes. Más seguro y potente que usar constantes `static final`.

## Enum básico

```java
public enum DiaSemana {
    LUNES, MARTES, MIERCOLES, JUEVES, VIERNES, SABADO, DOMINGO
}

// Uso
DiaSemana hoy = DiaSemana.LUNES;
System.out.println(hoy.name());    // "LUNES"
System.out.println(hoy.ordinal()); // 0 (posición)
```

## Enum con atributos y métodos

```java
public enum Planeta {
    MERCURIO(3.303e+23, 2.4397e6),
    VENUS   (4.869e+24, 6.0518e6),
    TIERRA  (5.976e+24, 6.37814e6);

    private final double masa;     // kg
    private final double radio;    // metros

    // Constructor (siempre privado)
    Planeta(double masa, double radio) {
        this.masa = masa;
        this.radio = radio;
    }

    public double getMasa() { return masa; }
    public double getRadio() { return radio; }

    static final double G = 6.67300E-11;

    public double gravedadSuperficial() {
        return G * masa / (radio * radio);
    }
}

// Uso
System.out.println(Planeta.TIERRA.gravedadSuperficial()); // ~9.8
```

## Enum con métodos abstractos

```java
public enum Operacion {
    SUMA {
        @Override
        public double aplicar(double a, double b) { return a + b; }
    },
    RESTA {
        @Override
        public double aplicar(double a, double b) { return a - b; }
    },
    MULTIPLICACION {
        @Override
        public double aplicar(double a, double b) { return a * b; }
    };

    public abstract double aplicar(double a, double b);
}

// Uso
double resultado = Operacion.SUMA.aplicar(5, 3); // 8.0
```

## Enum e interfaces

```java
public interface Describible {
    String getDescripcion();
}

public enum Estado implements Describible {
    ACTIVO {
        @Override
        public String getDescripcion() { return "El registro está activo"; }
    },
    INACTIVO {
        @Override
        public String getDescripcion() { return "El registro está inactivo"; }
    };
}
```

## switch con enum

```java
DiaSemana dia = DiaSemana.SABADO;

// Clásico
switch (dia) {
    case SABADO:
    case DOMINGO:
        System.out.println("Fin de semana");
        break;
    default:
        System.out.println("Día laboral");
}

// Moderno (Java 14+)
String tipo = switch (dia) {
    case SABADO, DOMINGO -> "Fin de semana";
    default -> "Día laboral";
};
```

## Métodos útiles

```java
// Obtener todos los valores
DiaSemana[] dias = DiaSemana.values();

// Obtener por nombre (String → enum)
DiaSemana lunes = DiaSemana.valueOf("LUNES");

// Comparar
DiaSemana.LUNES.compareTo(DiaSemana.VIERNES); // negativo (LUNES antes)
DiaSemana.LUNES == DiaSemana.LUNES; // true (safe con enums)
```

## EnumSet y EnumMap

```java
import java.util.EnumSet;
import java.util.EnumMap;

// EnumSet — conjunto de enums, muy eficiente
EnumSet<DiaSemana> finDeSemana = EnumSet.of(DiaSemana.SABADO, DiaSemana.DOMINGO);
EnumSet<DiaSemana> laboral = EnumSet.complementOf(finDeSemana);

// EnumMap — mapa con claves enum
EnumMap<DiaSemana, String> horarios = new EnumMap<>(DiaSemana.class);
horarios.put(DiaSemana.LUNES, "9:00 - 18:00");
```

## ⚠️ Errores Comunes

> [!warning] `valueOf()` lanza IllegalArgumentException si el nombre no existe
> ```java
> DiaSemana.valueOf("lunes"); // ❌ case-sensitive
> DiaSemana.valueOf("LUNES"); // ✅
> ```

## 📌 Notas Rápidas
