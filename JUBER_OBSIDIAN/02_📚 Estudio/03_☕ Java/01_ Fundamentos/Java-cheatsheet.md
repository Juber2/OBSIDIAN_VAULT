---
tags:
  - Java
  - Fundamentos
  - Cheatsheet
Creado: 2026-09-08
Relacionado:
  - Java Fundamentos
  - Bases java
Palabras clave:
  - sintaxis, referencia rapida, colecciones, streams, records, switch
---

## 🧠 Para qué sirve esta nota

Referencia de consulta rápida. No enseña — recuerda. Si un tema no te queda claro acá, la nota completa está enlazada al final.

## Tipos primitivos

| Tipo | Bits | Rango | Default | Wrapper |
|---|---|---|---|---|
| `byte` | 8 | -128 a 127 | 0 | `Byte` |
| `short` | 16 | -32.768 a 32.767 | 0 | `Short` |
| `int` | 32 | ±2.147.483.647 | 0 | `Integer` |
| `long` | 64 | ±9,22×10¹⁸ | 0L | `Long` |
| `float` | 32 | ~7 dígitos | 0.0f | `Float` |
| `double` | 64 | ~15 dígitos | 0.0d | `Double` |
| `char` | 16 | 0 a 65.535 | carácter nulo | `Character` |
| `boolean` | 1* | true/false | false | `Boolean` |

> [!danger] Nunca uses `float`/`double` para dinero
> `0.1 + 0.2 != 0.3` en binario. Para importes se usa `BigDecimal` construido desde `String`.
> ```java
> new BigDecimal("0.1").add(new BigDecimal("0.2")); // 0.3 exacto
> new BigDecimal(0.1);                              // ❌ arrastra el error binario
> ```

## Declaración y literales

```java
int decimal   = 1_000_000;      // guion bajo como separador visual
int hex       = 0xFF;           // 255
int binario   = 0b1010;         // 10
long grande   = 9_000_000_000L; // la L es obligatoria
double d      = 1.5e3;          // 1500.0
var lista     = new ArrayList<String>(); // inferencia (solo variables locales)
final int MAX = 100;            // constante
```

## Strings

```java
String s = "hola";

s.length();            s.isEmpty();          s.isBlank();   // vacío o solo espacios
s.charAt(0);           s.indexOf("o");       s.contains("ol");
s.substring(1, 3);     s.replace("a", "e");  s.strip();     // strip > trim (Unicode)
s.toUpperCase();       s.split(",");         s.repeat(3);
s.equals(otro);        s.equalsIgnoreCase(otro);

String.join("-", "a", "b");                  // "a-b"
String.format("%s tiene %d", nombre, edad);
```

```java
// Text block (Java 15+)
String json = """
    { "id": 1, "nombre": "Ana" }
    """;
```

> [!warning] `==` compara referencias, no contenido
> Para comparar texto siempre `equals`. Con `==` a veces "funciona" por el string pool, y eso lo vuelve un bug intermitente.

## Control de flujo

```java
// switch expression (Java 14+): devuelve valor, no necesita break
String tipo = switch (codigo) {
    case 1, 2 -> "AHORRO";
    case 3    -> "CORRIENTE";
    default   -> { yield "DESCONOCIDO"; }   // yield para bloques
};

// pattern matching para instanceof (Java 16+)
if (obj instanceof String s && s.length() > 3) { ... }

// for-each
for (var item : coleccion) { ... }

// try-with-resources: cierra solo
try (var reader = Files.newBufferedReader(path)) { ... }
```

## Colecciones — qué usar

| Necesito | Uso | Orden | Duplicados | Null |
|---|---|---|---|---|
| Lista indexada, muchas lecturas | `ArrayList` | inserción | sí | sí |
| Muchas inserciones/borrados al medio | `LinkedList` | inserción | sí | sí |
| Sin duplicados, rápido | `HashSet` | ninguno | no | 1 null |
| Sin duplicados, ordenado | `TreeSet` | natural | no | no |
| Sin duplicados, orden de inserción | `LinkedHashSet` | inserción | no | 1 null |
| Clave → valor | `HashMap` | ninguno | claves no | 1 null key |
| Clave → valor ordenado | `TreeMap` | natural | claves no | no |
| Cola FIFO / pila | `ArrayDeque` | FIFO | sí | no |
| Acceso concurrente | `ConcurrentHashMap` | ninguno | claves no | no |

```java
List<String> l = new ArrayList<>();
l.add("a"); l.get(0); l.remove("a"); l.size(); l.contains("a");

Map<String, Integer> m = new HashMap<>();
m.put("a", 1);
m.getOrDefault("b", 0);
m.putIfAbsent("c", 3);
m.computeIfAbsent("d", k -> 4);
m.merge("a", 1, Integer::sum);    // suma si existe, inserta si no
m.forEach((k, v) -> ...);

// Inmutables (Java 9+): lanzan UnsupportedOperationException al modificar
List.of("a", "b");   Set.of(1, 2);   Map.of("k", 1);
```

## Streams

```java
lista.stream()
     .filter(p -> p.edad() > 18)
     .map(Persona::nombre)
     .sorted()
     .distinct()
     .limit(10)
     .toList();                   // Java 16+, devuelve lista inmutable
```

```java
// Reducción
.count()      .anyMatch(pred)      .allMatch(pred)      .findFirst()
.reduce(0, Integer::sum)
.mapToInt(Persona::edad).sum()
.mapToInt(Persona::edad).average()

// Agrupar y recolectar
.collect(Collectors.groupingBy(Persona::ciudad))
.collect(Collectors.groupingBy(Persona::ciudad, Collectors.counting()))
.collect(Collectors.toMap(Persona::id, Function.identity()))
.collect(Collectors.joining(", "))
.collect(Collectors.partitioningBy(p -> p.edad() >= 18))
```

> [!tip] Los streams son de un solo uso
> Una vez consumido (`toList()`, `count()`, …) no se puede reutilizar: hay que volver a crearlo desde la fuente.

## Optional

```java
Optional<Cuenta> o = repo.findById(id);

o.isPresent();                  o.isEmpty();
o.get();                        // ❌ evitar: lanza NoSuchElementException
o.orElse(cuentaDefault);        // evalúa el argumento SIEMPRE
o.orElseGet(() -> costoso());   // evalúa solo si está vacío ✅
o.orElseThrow(() -> new NotFoundException(id));
o.map(Cuenta::saldo).filter(s -> s.signum() > 0).ifPresent(...);
o.ifPresentOrElse(c -> ..., () -> ...);
```

## Records, enums y sealed

```java
// Record: portador de datos inmutable, con equals/hashCode/toString automáticos
public record CuentaDTO(String iban, BigDecimal saldo) {
    public CuentaDTO {                       // constructor compacto = validación
        if (saldo.signum() < 0) throw new IllegalArgumentException("saldo negativo");
    }
}

// Enum con estado y comportamiento
public enum Moneda {
    PEN("S/", 2), USD("$", 2);

    private final String simbolo;
    private final int decimales;

    Moneda(String simbolo, int decimales) {
        this.simbolo = simbolo;
        this.decimales = decimales;
    }

    public String simbolo() { return simbolo; }
}

// Sealed: jerarquía cerrada, el compilador exige cubrir todos los casos
public sealed interface Evento permits Deposito, Retiro {}
```

## Excepciones

```java
try {
    ...
} catch (IOException | SQLException e) {   // multi-catch
    log.error("fallo al leer", e);          // SIEMPRE propagar la causa
    throw new AppException("mensaje de negocio", e);
} finally {
    // se ejecuta siempre, salvo System.exit
}
```

| | Checked | Unchecked |
|---|---|---|
| Hereda de | `Exception` | `RuntimeException` |
| El compilador obliga a manejarla | sí | no |
| Cuándo usarla | error esperado y recuperable | bug o violación de contrato |
| Ejemplos | `IOException` | `NullPointerException`, `IllegalArgumentException` |

## Concurrencia — atajos

```java
var pool    = Executors.newFixedThreadPool(4);
var virtual = Executors.newVirtualThreadPerTaskExecutor();   // Java 21

CompletableFuture.supplyAsync(() -> servicio.buscar(id))
    .thenApply(Dto::from)
    .exceptionally(ex -> Dto.vacio())
    .join();

// Estructuras seguras para concurrencia
new ConcurrentHashMap<>();   new CopyOnWriteArrayList<>();   new AtomicInteger();
```

## java.time

```java
LocalDate.now();            LocalDateTime.now();      Instant.now();   // UTC
LocalDate.of(2026, 9, 8);   LocalDate.parse("2026-09-08");

fecha.plusDays(3).minusMonths(1).withDayOfMonth(1);
ChronoUnit.DAYS.between(a, b);
Duration.ofMinutes(30);     Period.between(a, b);
ZonedDateTime.now(ZoneId.of("America/Lima"));
fecha.format(DateTimeFormatter.ofPattern("dd/MM/yyyy"));
```

> [!tip] Regla para backend
> Persistí y transportá en **UTC** (`Instant`). Convertí a zona local solo al presentar.

## Modificadores de acceso

| Modificador | Misma clase | Mismo paquete | Subclase | Todos |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(default)* | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

## Comandos de terminal

```bash
java --version
javac App.java && java App          # compilar y ejecutar
java App.java                       # ejecución directa (Java 11+)
jar --create --file app.jar -C out .
java -jar app.jar
jshell                              # REPL interactivo
```

```bash
jps -l                              # procesos Java corriendo
jstack <pid>                        # volcado de hilos
jmap -histo <pid>                   # histograma de objetos en heap
java -XX:+PrintFlagsFinal -version  # flags efectivos de la JVM
```

## 🔗 Relaciones
- **Base**: [[Bases java]], [[Java Fundamentos]]
- **Profundizar**: [[Java Intermedio]], [[Java Avanzado]]
- **Errores frecuentes**: [[Java-errores-comunes]]

## 📌 Notas Rápidas
- Esta nota es de consulta, no de estudio: si tenés que leerla entera para entender algo, la nota que necesitás es otra.
- `var` solo funciona en variables locales — no en campos, parámetros ni tipos de retorno.
- Preferí colecciones inmutables (`List.of`) cuando no vas a modificar: evitan bugs de mutación compartida.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

