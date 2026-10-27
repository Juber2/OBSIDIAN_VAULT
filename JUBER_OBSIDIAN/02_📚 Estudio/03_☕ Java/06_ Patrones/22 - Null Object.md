---
tags:
  - Java
  - Patrones
  - No-GoF
  - Comportamiento
Creado: 2026-09-10
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - null object, objeto nulo, NullPointerException, Optional, comportamiento neutro
---

## 🧠 Concepto

En vez de devolver `null`, devolvés un objeto **que implementa la misma interfaz y no hace nada**. El llamador deja de tener que preguntar.

**Cuándo usarlo:** cuando "no hay nada" es un caso normal y el comportamiento correcto ante esa ausencia es no hacer nada o devolver un valor neutro.

## El problema

```java
// ❌ Cada llamador tiene que acordarse de preguntar
public class ServicioTransferencia {

    private final Auditor auditor;      // puede ser null si la auditoría está desactivada

    public void transferir(...) {
        ...
        if (auditor != null) {           // ← esta pregunta se repite en cada método
            auditor.registrar(evento);
        }
    }

    public void reversar(...) {
        ...
        if (auditor != null) auditor.registrar(evento);      // otra vez
    }
}
```

El día que alguien olvide el `if`, hay un `NullPointerException` en producción.

## Implementación

```java
public interface Auditor {
    void registrar(EventoAuditoria evento);
    List<EventoAuditoria> consultar(Filtro f);
}

public class AuditorEnBase implements Auditor {
    @Override public void registrar(EventoAuditoria e) { repo.insertar(e); }
    @Override public List<EventoAuditoria> consultar(Filtro f) { return repo.buscar(f); }
}

// El objeto nulo: misma interfaz, comportamiento neutro
public class AuditorNulo implements Auditor {
    @Override public void registrar(EventoAuditoria e) { /* no hace nada */ }
    @Override public List<EventoAuditoria> consultar(Filtro f) { return List.of(); }
}
```

```java
public class ServicioTransferencia {

    private final Auditor auditor;

    public ServicioTransferencia(Auditor auditor) {
        this.auditor = Objects.requireNonNullElseGet(auditor, AuditorNulo::new);
    }

    public void transferir(...) {
        ...
        auditor.registrar(evento);        // ✅ sin if, sin NPE posible
    }
}
```

> [!tip] El objeto nulo debe devolver valores neutros, no `null`
> `consultar()` devuelve `List.of()`, no `null`. Si devolviera `null`, moviste el problema un nivel más abajo en vez de eliminarlo.
> Neutro significa: lista vacía, `Dinero.cero()`, string vacío, `Optional.empty()`. Nunca `null`.

## Con Spring

```java
@Component
@ConditionalOnProperty(name = "auditoria.habilitada", havingValue = "true")
public class AuditorEnBase implements Auditor { ... }

@Component
@ConditionalOnMissingBean(Auditor.class)          // solo si no hay otro
public class AuditorNulo implements Auditor { ... }
```

El servicio recibe siempre un `Auditor` funcional. La configuración decide cuál, y el código de negocio ni se entera.

## Otro caso: el enum con constante nula

```java
public enum PoliticaDeDescuento {

    SIN_DESCUENTO {
        @Override public Dinero aplicar(Dinero precio) { return precio; }   // neutro
    },
    CLIENTE_FRECUENTE {
        @Override public Dinero aplicar(Dinero precio) { return precio.porcentaje(0.90); }
    },
    PROMOCION {
        @Override public Dinero aplicar(Dinero precio) { return precio.porcentaje(0.75); }
    };

    public abstract Dinero aplicar(Dinero precio);
}
```

```java
Dinero final = cliente.politica().aplicar(precio);   // nunca hay que chequear null
```

`SIN_DESCUENTO` es un objeto nulo: cumple el contrato y no cambia nada.

## 🆚 Null Object vs `Optional`

Se resuelven casos distintos y conviene no mezclarlos.

| | Null Object | `Optional` |
|---|---|---|
| Qué expresa | *"no hay nada y está bien, seguí"* | *"puede no haber, decidí vos"* |
| Quién decide | el objeto | el llamador |
| Dónde va | **colaboradores** (servicios, políticas) | **valores de retorno** de búsquedas |
| Uso típico | `Auditor`, `Notificador`, `Cache` | `repo.buscarPorId(id)` |

```java
// ✅ Optional: la ausencia es información que el llamador debe manejar
Optional<Cuenta> cuenta = repo.porNumero(numero);
cuenta.orElseThrow(() -> new CuentaNoEncontradaException(numero));

// ✅ Null Object: la ausencia no cambia el flujo
auditor.registrar(evento);       // haya o no auditoría, el negocio sigue igual
```

> [!danger] `Optional` no va como campo ni como parámetro
> ```java
> private Optional<Auditor> auditor;                   // ❌
> public void transferir(Optional<Auditor> auditor)    // ❌
> ```
> `Optional` fue diseñado para **tipos de retorno**. Como campo no es serializable y agrega una capa que hay que desenvolver en cada uso; como parámetro obliga al llamador a construir un `Optional` para nada.
> Para colaboradores ausentes, el objeto nulo es la respuesta.

## 🎯 Cuándo NO usarlo

| Situación | Por qué no |
|---|---|
| La ausencia **es** un error | ocultarla hace que el bug pase inadvertido |
| El llamador necesita reaccionar distinto | usá `Optional` y que decida |
| No existe un comportamiento neutro razonable | forzarlo produce resultados raros |

> [!danger] El riesgo real: esconder un error
> ```java
> // ❌ Si el repositorio devuelve una CuentaNula cuando no encuentra la cuenta...
> Cuenta cuenta = repo.porNumero(numero);      // devuelve CuentaNula
> cuenta.debitar(monto);                        // no hace nada
> // La transferencia "funciona" y el dinero nunca se movió 💥
> ```
> Para una búsqueda que puede no encontrar nada, va `Optional`. El objeto nulo es para **colaboradores opcionales**, no para entidades que deberían existir.

## 📌 Notas Rápidas
- La pregunta que decide: **¿el llamador tiene que hacer algo distinto si no hay nada?** Si sí, `Optional`. Si no, objeto nulo.
- Ya lo usás sin saberlo: `Collections.emptyList()`, `Optional.empty()`, `NOPLogger` de SLF4J y `NullOutputStream` son todos objetos nulos.
- El objeto nulo tiene que devolver **valores neutros**, nunca `null`. Si devuelve `null`, no resolviste nada.
- Nunca lo uses para entidades que deberían existir: convierte un error ruidoso en un fallo silencioso, que siempre es peor.
- Con Spring, `@ConditionalOnMissingBean` lo hace casi automático: una implementación real cuando la funcionalidad está activa, la nula cuando no.
- No es GoF: viene de *Pattern Languages of Program Design*.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
