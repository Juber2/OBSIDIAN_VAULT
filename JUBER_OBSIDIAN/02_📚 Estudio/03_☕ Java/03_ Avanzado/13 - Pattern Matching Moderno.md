---
tags:
  - Java
  - Avanzado
  - Lenguaje
Creado: 2026-09-08
Relacionado:
  - Java Avanzado
  - 09 - Records y Sealed Classes
Palabras clave:
  - pattern matching, switch, record pattern, sealed, exhaustividad, guardas, when
---

## 🧠 Concepto

El *pattern matching* permite **preguntar por la forma de un dato y extraer sus partes en una sola expresión**. Java lo fue incorporando por etapas desde la versión 16, y combinado con `record` y `sealed` cambia bastante cómo se modela el dominio.

| Versión | Qué llegó |
|---|---|
| 16 | `instanceof` con patrón |
| 17 | clases `sealed` |
| 21 | patrones en `switch`, patrones de `record`, `null` en `switch` |

## 1️⃣ `instanceof` con patrón

```java
// ❌ Antes: comprobar, castear, usar
if (obj instanceof String) {
    String s = (String) obj;
    if (s.length() > 3) { ... }
}

// ✅ Ahora: una sola expresión
if (obj instanceof String s && s.length() > 3) { ... }
```

> [!tip] El alcance de la variable es exactamente donde tiene sentido
> `s` existe dentro del `if` y también en el resto de la condición gracias al cortocircuito de `&&`. Fuera de ahí, no compila. El compilador razona sobre el flujo.

```java
// También funciona negado, con el alcance invertido
public void procesar(Object obj) {
    if (!(obj instanceof Movimiento m)) {
        throw new TipoInvalidoException(obj);
    }
    // acá m ES visible: el compilador sabe que si llegamos, el patrón se cumplió
    registrar(m);
}
```

## 2️⃣ Patrones en `switch`

```java
public String describir(Object obj) {
    return switch (obj) {
        case Integer i when i < 0  -> "entero negativo: " + i;
        case Integer i             -> "entero: " + i;
        case String s              -> "texto de " + s.length() + " caracteres";
        case int[] arr             -> "array de " + arr.length;
        case null                  -> "nulo";          // ← se puede manejar explícitamente
        default                    -> "otra cosa";
    };
}
```

> [!warning] El orden importa: gana el primer caso que coincide
> ```java
> case Integer i             -> "entero";
> case Integer i when i < 0  -> "negativo";     // ❌ inalcanzable: no compila
> ```
> El compilador detecta el caso dominado y falla. Los patrones más específicos van primero.

> [!danger] `switch` sin `case null` sigue lanzando `NullPointerException`
> El comportamiento histórico se mantuvo por compatibilidad. Lo nuevo es que ahora **podés** manejar el nulo explícitamente con `case null`. Si no lo hacés, un `switch` sobre una referencia nula falla igual que siempre.

### Guardas con `when`

```java
public Decision evaluar(Operacion op) {
    return switch (op) {
        case Transferencia t when t.monto().compareTo(LIMITE_ALTO) > 0
                                        -> Decision.revisar("monto elevado");
        case Transferencia t when t.destinoEsNuevo()
                                        -> Decision.desafiar(OTP);
        case Transferencia t            -> Decision.permitir();
        case Retiro r when r.esCajeroAjeno()
                                        -> Decision.permitirConComision();
        case Retiro r                   -> Decision.permitir();
        case Deposito d                 -> Decision.permitir();
    };
}
```

> [!tip] Las guardas reemplazan cadenas de `if` anidados
> El mismo código con `if/else` sería tres niveles de anidamiento y una variable temporal por rama. Acá cada línea es una regla legible de arriba abajo.

## 3️⃣ Patrones de `record`

Descomponen el objeto en sus componentes directamente en el patrón.

```java
public sealed interface Evento
    permits Deposito, Retiro, Transferencia, Reversa { }

public record Deposito(CuentaId cuenta, Dinero monto) implements Evento { }
public record Retiro(CuentaId cuenta, Dinero monto, Canal canal) implements Evento { }
public record Transferencia(CuentaId origen, CuentaId destino, Dinero monto) implements Evento { }
public record Reversa(UUID asientoOriginal, String motivo) implements Evento { }
```

```java
public List<Apunte> apuntesDe(Evento evento) {
    return switch (evento) {

        // Los componentes se extraen en el propio patrón
        case Deposito(var cuenta, var monto) ->
            List.of(new Apunte(CAJA, monto.negado()), new Apunte(cuenta, monto));

        case Retiro(var cuenta, var monto, Canal.CAJERO) ->      // literal en el patrón
            List.of(new Apunte(cuenta, monto.negado()),
                    new Apunte(CAJA, monto),
                    new Apunte(COMISIONES, COMISION_CAJERO));

        case Retiro(var cuenta, var monto, var canal) ->
            List.of(new Apunte(cuenta, monto.negado()), new Apunte(CAJA, monto));

        case Transferencia(var origen, var destino, var monto) ->
            List.of(new Apunte(origen, monto.negado()), new Apunte(destino, monto));

        case Reversa(var original, var motivo) ->
            invertirApuntesDe(original);
    };
}
```

### Anidados

```java
// Se descompone en profundidad, en una sola expresión
case Transferencia(var origen, var destino, Dinero(var monto, Moneda.USD))
        when monto.compareTo(UMBRAL_USD) > 0 ->
    reportarOperacionEnDivisas(origen, destino, monto);
```

> [!tip] Esto reemplaza al patrón Visitor completo
> El Visitor de GoF existe porque Java no tenía forma de despachar según el tipo concreto sin romper el encapsulamiento. Con `sealed` + patrones de `record`, el mismo problema se resuelve en un `switch` legible y con verificación de exhaustividad del compilador.

## 🔒 Exhaustividad

El aporte más valioso, y el que menos se aprovecha.

```java
public sealed interface Evento permits Deposito, Retiro, Transferencia, Reversa { }

// Sin default: el compilador EXIGE cubrir todos los casos
public String describir(Evento e) {
    return switch (e) {
        case Deposito d      -> "depósito";
        case Retiro r        -> "retiro";
        case Transferencia t -> "transferencia";
        // ❌ ERROR DE COMPILACIÓN: falta cubrir Reversa
    };
}
```

> [!danger] Nunca pongas `default` en un `switch` sobre un tipo sellado
> ```java
> default -> "desconocido";     // ❌ desactiva la verificación de exhaustividad
> ```
> El `default` hace que el compilador deje de avisarte. El día que agregues un tipo nuevo al `permits`, **todos los `switch` seguirán compilando** y caerán silenciosamente en el `default`.
>
> Sin `default`, agregar un tipo rompe la compilación en cada lugar que hay que actualizar. **Eso es exactamente lo que querés**: el compilador te lleva de la mano por todos los puntos a revisar.

> [!tip] Es la razón principal para usar `sealed`
> Modelar el dominio con jerarquías cerradas convierte "me olvidé de manejar un caso nuevo" —un bug de producción— en un error de compilación.

## 🎯 Cuándo usarlo y cuándo no

```java
// ❌ Pattern matching donde corresponde polimorfismo
public double area(Figura f) {
    return switch (f) {
        case Circulo c   -> Math.PI * c.radio() * c.radio();
        case Cuadrado cu -> cu.lado() * cu.lado();
    };
}

// ✅ Polimorfismo: el comportamiento pertenece al tipo
public sealed interface Figura permits Circulo, Cuadrado {
    double area();
}
```

| Usá polimorfismo | Usá pattern matching |
|---|---|
| El comportamiento pertenece al tipo | la operación es externa al dominio |
| Los tipos evolucionan más que las operaciones | las operaciones evolucionan más que los tipos |
| Cada tipo sabe hacerlo | la lógica depende de combinaciones |
| El dominio te pertenece | trabajás con tipos ajenos |

> [!tip] La pregunta que decide
> **¿La operación pertenece conceptualmente al objeto?** Calcular el área es del círculo. Serializar a JSON, mapear a un DTO o generar apuntes contables desde un evento, no: son operaciones externas que dependen del tipo pero no le pertenecen.

## 💡 Dónde rinde en un backend

```java
// 1 · Manejo de errores por tipo
@ExceptionHandler
public ProblemDetail manejar(Exception ex) {
    return switch (ex) {
        case SaldoInsuficienteException e ->
            problema(422, "Saldo insuficiente", Map.of("disponible", e.disponible()));
        case CuentaNoEncontradaException e -> problema(404, "Cuenta no encontrada");
        case LimiteExcedidoException e when e.esDiario() -> problema(422, "Límite diario");
        case LimiteExcedidoException e -> problema(422, "Límite mensual");
        case AccessDeniedException e -> problema(403, "Sin permiso");
        default -> problema(500, "Error interno");
    };
}
```

```java
// 2 · Procesar eventos de dominio
@KafkaListener(topics = "eventos")
public void alRecibir(EventoDominio evento) {
    switch (evento) {
        case MovimientoRegistrado(var id, var cuenta, var monto, _, _, _) ->
            proyeccion.actualizar(cuenta, monto);
        case TransferenciaCompletada(var id, var origen, var destino, var monto) ->
            notificador.enviarA(origen, destino, monto);
        case TransferenciaReversada(var original, var reversa, var motivo) ->
            notificador.avisarReversa(original, motivo);
    }
}
```

```java
// 3 · Máquinas de estado
public EstadoSaga siguiente(EstadoSaga actual, ResultadoPaso resultado) {
    return switch (actual) {
        case INICIADA          -> resultado.exitoso() ? FONDOS_RESERVADOS : FALLIDA;
        case FONDOS_RESERVADOS -> resultado.exitoso() ? DEBITADA : COMPENSANDO;
        case DEBITADA          -> switch (resultado.tipo()) {
                                      case EXITO   -> ACREDITADA;
                                      case RECHAZO -> COMPENSANDO;
                                      case TIMEOUT -> EN_REVISION;
                                  };
        case ACREDITADA        -> COMPLETADA;
        case COMPENSANDO       -> FALLIDA;
        case COMPLETADA, FALLIDA, EN_REVISION -> actual;
    };
}
```

> [!tip] La máquina de estados de una SAGA es el caso ideal
> Estados cerrados, transiciones explícitas y verificación de exhaustividad: si mañana agregás un estado, el compilador te lleva a todos los `switch` que hay que actualizar — ver [[06 - SAGA]].

## 🧹 Patrones sin nombre

```java
// Java 21+ : _ para lo que no usás
case MovimientoRegistrado(var id, var cuenta, var monto, _, _, _) ->
    proyeccion.actualizar(cuenta, monto);
```

Evita nombrar variables que no se usan y deja explícito qué componentes importan.

## 🧪 Tests

```java
@Test
void el_switch_cubre_todos_los_eventos() {
    // Si se agrega un tipo al sealed sin actualizar el switch, esto NO COMPILA.
    // El test documenta la intención; el compilador la hace cumplir.
    for (var evento : eventosDeTodosLosTipos()) {
        assertDoesNotThrow(() -> servicio.apuntesDe(evento));
    }
}

@Test
void el_retiro_por_cajero_ajeno_genera_apunte_de_comision() {
    var apuntes = servicio.apuntesDe(new Retiro(cuenta, Dinero.de("100", PEN), Canal.CAJERO));
    assertEquals(3, apuntes.size());
    assertTrue(apuntes.stream().anyMatch(a -> a.cuenta().equals(COMISIONES)));
}
```

## ⚠️ Errores Comunes

> [!warning] Los que se ven al adoptarlo
> - **`default` en un `switch` sobre tipo sellado**: mata la exhaustividad.
> - **Patrón general antes que el específico**: no compila (caso dominado).
> - **Olvidar `case null`** y asumir que el `switch` ya no falla con nulos.
> - **Usar pattern matching donde corresponde polimorfismo.**
> - **`switch` gigantes** con veinte casos: probablemente falta modelar mejor.
> - **`sealed` sin aprovechar la exhaustividad**: se pierde el beneficio principal.

## 📌 Notas Rápidas
- El valor real no es la sintaxis más corta: es la **exhaustividad verificada por el compilador**. Convierte "me olvidé un caso" en un error de compilación.
- La combinación `sealed` + `record` + patrones de `record` es lo más cercano que tiene Java a los tipos algebraicos de lenguajes funcionales, y encaja muy bien para modelar eventos de dominio y máquinas de estado.
- La regla para decidir entre polimorfismo y pattern matching: **¿el comportamiento pertenece al tipo?** Si sí, método. Si es una operación externa, `switch`.
- En una entrevista, explicar por qué **no** poner `default` en un `switch` sellado demuestra que entendiste para qué sirve la característica.

## 🔗 Relaciones
- **Módulo**: [[Java Avanzado]] — última nota del módulo
- **Anterior**: [[12 - Structured Concurrency]]
- **Base**: [[09 - Records y Sealed Classes]]
- **Se conecta con**: [[06 - SAGA]], [[08 - Event Sourcing]], [[08 - Manejo de Errores]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

