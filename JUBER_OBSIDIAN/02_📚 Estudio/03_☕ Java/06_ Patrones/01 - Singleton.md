---
tags:
  - Java
  - Patrones
  - Creacional
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - singleton, instancia única, lazy, eager, double-checked locking, enum singleton
---

## 🧠 Concepto

Garantiza que una clase tenga **una sola instancia** y proporciona un punto de acceso global a ella.

**Cuándo usarlo:** Configuración global, conexión a BD, loggers, cachés.

## Implementaciones

### 1. Eager (temprana) — más simple

```java
public class Configuracion {
    private static final Configuracion INSTANCIA = new Configuracion(); // se crea al cargar la clase

    private Configuracion() {} // constructor privado

    public static Configuracion getInstance() {
        return INSTANCIA;
    }
}
```

### 2. Lazy (perezosa) — solo cuando se necesita

```java
public class Configuracion {
    private static Configuracion instancia;

    private Configuracion() {}

    public static Configuracion getInstance() {
        if (instancia == null) {
            instancia = new Configuracion(); // ❌ NO thread-safe
        }
        return instancia;
    }
}
```

### 3. Thread-safe con double-checked locking (recomendado)

```java
public class Configuracion {
    private static volatile Configuracion instancia; // volatile es clave

    private Configuracion() {}

    public static Configuracion getInstance() {
        if (instancia == null) {
            synchronized (Configuracion.class) {
                if (instancia == null) { // segundo check dentro del lock
                    instancia = new Configuracion();
                }
            }
        }
        return instancia;
    }
}
```

### 4. Holder idiom (más elegante, lazy y thread-safe)

```java
public class Configuracion {
    private Configuracion() {}

    private static class Holder {
        private static final Configuracion INSTANCIA = new Configuracion();
    }

    public static Configuracion getInstance() {
        return Holder.INSTANCIA; // la clase Holder se carga solo cuando se llama este método
    }
}
```

### 5. Enum Singleton (más robusto — protegido de Reflection y serialización)

```java
public enum Configuracion {
    INSTANCIA;

    private String baseDatos;

    public String getBaseDatos() { return baseDatos; }
    public void setBaseDatos(String bd) { this.baseDatos = bd; }
}

// Uso
Configuracion.INSTANCIA.setBaseDatos("mi-db");
```

## Singleton en Spring

```java
// En Spring, los @Bean son Singleton por defecto
@Bean
public MiServicio miServicio() {
    return new MiServicio();
}

// Explícito
@Component
@Scope("singleton") // default
public class MiServicio { }
```

## ⚠️ Errores Comunes

> [!warning] Singleton sin `volatile` no es thread-safe
> Sin `volatile`, el compilador puede reordenar instrucciones y otro hilo puede ver una instancia parcialmente construida.

> [!warning] Singleton rompe testabilidad
> Es difícil de mockear. Preferir inyección de dependencias (Spring) sobre Singleton manual.

## 📌 Notas Rápidas
