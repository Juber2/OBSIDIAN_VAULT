---
tags:
  - Java
  - Patrones
  - Comportamiento
Creado: 2026-09-10
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - state, maquina de estados, transiciones, enum, comportamiento segun estado
---

## 🧠 Concepto

Permite que un objeto **cambie su comportamiento cuando cambia su estado interno**. Desde afuera parece que el objeto cambió de clase.

**Cuándo usarlo:** el comportamiento depende de un estado que cambia, y las transiciones tienen reglas. Reemplaza los `switch (estado)` repartidos por todo el código.

## El problema que resuelve

```java
// ❌ El switch se repite en cada método y crece con cada estado nuevo
public class Transferencia {
    private String estado;

    public void confirmar() {
        switch (estado) {
            case "INICIADA": estado = "CONFIRMADA"; break;
            case "CONFIRMADA": throw new IllegalStateException("ya confirmada");
            case "CANCELADA": throw new IllegalStateException("está cancelada");
        }
    }

    public void cancelar() {
        switch (estado) { ... }      // otro switch con las mismas ramas
    }
}
```

## Implementación

```java
public interface EstadoTransferencia {
    EstadoTransferencia confirmar(Transferencia t);
    EstadoTransferencia cancelar(Transferencia t);
    default boolean esFinal() { return false; }
}

public class Iniciada implements EstadoTransferencia {
    public EstadoTransferencia confirmar(Transferencia t) {
        t.registrarAsiento();
        return new Confirmada();
    }
    public EstadoTransferencia cancelar(Transferencia t) {
        t.liberarReserva();
        return new Cancelada();
    }
}

public class Confirmada implements EstadoTransferencia {
    public EstadoTransferencia confirmar(Transferencia t) {
        throw new TransicionInvalidaException("ya está confirmada");
    }
    public EstadoTransferencia cancelar(Transferencia t) {
        t.registrarReversa();                 // cancelar una confirmada = reversar
        return new Reversada();
    }
}

public class Cancelada implements EstadoTransferencia {
    public EstadoTransferencia confirmar(Transferencia t) {
        throw new TransicionInvalidaException("está cancelada");
    }
    public EstadoTransferencia cancelar(Transferencia t) { return this; }  // idempotente
    public boolean esFinal() { return true; }
}
```

```java
public class Transferencia {
    private EstadoTransferencia estado = new Iniciada();

    public void confirmar() { estado = estado.confirmar(this); }
    public void cancelar()  { estado = estado.cancelar(this); }
}
```

> [!tip] Cada estado sabe a cuál puede pasar
> Las reglas de transición dejan de estar dispersas en `switch` y quedan en un solo lugar por estado. Agregar un estado nuevo es una clase nueva, no editar seis métodos.

## Con enum — la versión que más se usa en Java

```java
public enum EstadoTransferencia {

    INICIADA {
        @Override public EstadoTransferencia confirmar(Transferencia t) {
            t.registrarAsiento();
            return CONFIRMADA;
        }
        @Override public EstadoTransferencia cancelar(Transferencia t) {
            t.liberarReserva();
            return CANCELADA;
        }
    },

    CONFIRMADA {
        @Override public EstadoTransferencia confirmar(Transferencia t) {
            throw new TransicionInvalidaException(this, "confirmar");
        }
        @Override public EstadoTransferencia cancelar(Transferencia t) {
            t.registrarReversa();
            return REVERSADA;
        }
    },

    CANCELADA, REVERSADA;      // finales: usan la implementación por defecto

    public EstadoTransferencia confirmar(Transferencia t) {
        throw new TransicionInvalidaException(this, "confirmar");
    }
    public EstadoTransferencia cancelar(Transferencia t) {
        throw new TransicionInvalidaException(this, "cancelar");
    }
}
```

> [!tip] El enum con cuerpo por constante es lo mejor de los dos mundos
> Tenés el comportamiento por estado **y** un valor persistible directo en la base (`VARCHAR`), serializable y comparable. Es la forma habitual en un backend Java.
> El costo: no podés agregar estados sin recompilar, lo cual casi siempre está bien.

## Con `sealed` y pattern matching — la versión moderna

```java
public sealed interface Estado permits Iniciada, Confirmada, Cancelada, Reversada { }

public Estado siguiente(Estado actual, Evento evento) {
    return switch (actual) {
        case Iniciada i   -> switch (evento) {
                                 case CONFIRMAR -> new Confirmada();
                                 case CANCELAR  -> new Cancelada();
                             };
        case Confirmada c -> switch (evento) {
                                 case CANCELAR  -> new Reversada();
                                 case CONFIRMAR -> throw new TransicionInvalidaException();
                             };
        case Cancelada c, Reversada r -> actual;    // finales
    };
}
```

Con tipos sellados el compilador **exige cubrir todos los estados**: si agregás uno, el `switch` deja de compilar en cada lugar que hay que actualizar. Ver [[13 - Pattern Matching Moderno]].

## Persistir el estado

```sql
CREATE TABLE saga_transferencia (
    id          UUID PRIMARY KEY,
    estado      VARCHAR(30) NOT NULL,      -- el nombre de la constante del enum
    ...
);
```

```java
@Entity
public class SagaEntity {
    @Enumerated(EnumType.STRING)           // ⚠️ STRING, nunca ORDINAL
    private EstadoTransferencia estado;
}
```

> [!danger] `EnumType.ORDINAL` guarda la posición, no el nombre
> Si mañana alguien reordena las constantes del enum o inserta una en el medio, **todos los registros existentes pasan a significar otra cosa**. No hay error: los datos simplemente quedan mal.
> Siempre `EnumType.STRING`.

## 🎯 Cuándo NO usarlo

| Situación | Alternativa |
|---|---|
| Dos estados sin reglas de transición | un `boolean` |
| El estado no cambia el comportamiento, solo se muestra | un enum simple |
| Las transiciones son un grafo enorme | una biblioteca de máquina de estados |
| El estado vive fuera del objeto | orquestador explícito |

## 📌 Notas Rápidas
- Es el patrón que está detrás de la SAGA de [[06 - SAGA]] y del Circuit Breaker de [[05 - Circuit Breaker - Resilience4j]]: `CLOSED`/`OPEN`/`HALF_OPEN` son estados con transiciones definidas.
- La señal de que lo necesitás: el mismo `switch (estado)` aparece en tres o más métodos, y agregar un estado obliga a tocarlos todos.
- Distinto de [[11 - Strategy]]: en Strategy **el cliente elige** el algoritmo; en State **el objeto decide** su próximo estado según las reglas internas.
- Para persistir: `@Enumerated(EnumType.STRING)` siempre. El `ORDINAL` es una bomba de tiempo.
- La versión con `enum` es la que más vas a ver y escribir en Java. La versión con clases separadas conviene cuando cada estado tiene mucho comportamiento propio.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
