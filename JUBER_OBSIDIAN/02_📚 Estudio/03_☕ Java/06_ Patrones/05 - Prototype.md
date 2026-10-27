---
tags:
  - Java
  - Patrones
  - Creacional
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - prototype, clone, copia, Cloneable, deep copy, shallow copy
---

## 🧠 Concepto

Crea nuevos objetos **copiando** un objeto existente (prototipo). Útil cuando la creación desde cero es costosa.

**Cuándo usarlo:** Objetos costosos de inicializar, cuando la mayoría de campos son iguales y solo cambian algunos.

## Implementación con Cloneable

```java
public class Configuracion implements Cloneable {
    private String servidor;
    private int puerto;
    private List<String> plugins; // campo de referencia

    public Configuracion(String servidor, int puerto) {
        this.servidor = servidor;
        this.puerto = puerto;
        this.plugins = new ArrayList<>();
    }

    // Shallow copy — copia primitivos, pero comparte referencias
    @Override
    public Configuracion clone() {
        try {
            return (Configuracion) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

    // Deep copy — copia todo, incluyendo objetos referenciados
    public Configuracion deepCopy() {
        Configuracion copia = new Configuracion(this.servidor, this.puerto);
        copia.plugins = new ArrayList<>(this.plugins); // nueva lista
        return copia;
    }
}

// Uso
Configuracion original = new Configuracion("localhost", 8080);
original.getPlugins().add("plugin1");

Configuracion copia = original.deepCopy();
copia.setServidor("produccion.server.com");
// original no cambia
```

## Implementación sin Cloneable (alternativa más limpia)

```java
public class Perfil {
    private String nombre;
    private List<String> permisos;

    // Copy constructor
    public Perfil(Perfil otro) {
        this.nombre = otro.nombre;
        this.permisos = new ArrayList<>(otro.permisos);
    }

    // Factory method de copia
    public static Perfil copiar(Perfil original) {
        return new Perfil(original);
    }
}
```

## Shallow vs Deep copy

| | Shallow Copy | Deep Copy |
|--|-------------|-----------|
| Primitivos | ✅ Copia el valor | ✅ Copia el valor |
| Strings | ✅ Inmutables, compartir es seguro | ✅ |
| Objetos referenciados | ⚠️ Comparte la referencia | ✅ Crea nuevos objetos |
| Rendimiento | Más rápido | Más lento |
| Seguridad | Cambios en uno afectan al otro | Completamente independientes |

## 📌 Notas Rápidas
- `Cloneable` es una interfaz "marcadora" (sin métodos) — sin implementarla, `Object.clone()` lanza `CloneNotSupportedException`, un diseño criticado del propio JDK.
- En Java moderno, un copy constructor o un método factory de copia (sin `Cloneable`) suele ser más limpio y explícito que usar `clone()`.
- Records (Java 16+) simplifican mucho este patrón para DTOs inmutables: crear una copia con un campo cambiado es trivial con un constructor compacto o un método `with...`.
- Poco usado como patrón "puro" en backends modernos — más relevante el concepto de shallow vs deep copy, que sí aparece seguido (ej. copiar una entidad JPA con sus colecciones).

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

