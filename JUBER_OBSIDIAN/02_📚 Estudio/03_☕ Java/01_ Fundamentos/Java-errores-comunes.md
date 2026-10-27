---
tags:
  - Java
  - Fundamentos
  - Errores
Creado: 2026-09-08
Relacionado:
  - Java Fundamentos
  - Java-cheatsheet
Palabras clave:
  - NullPointerException, ConcurrentModification, equals, hashCode, BigDecimal, N+1
---

## 🧠 Para qué sirve esta nota

Catálogo de los errores que **todo el mundo comete** en Java, con el síntoma, la causa real y el arreglo. Ordenados de "te pasa el primer día" a "te pasa en producción a las 3am".

---

# Nivel 1 — Sintaxis y objetos

## NullPointerException

**Síntoma:** `Cannot invoke "String.length()" because "s" is null`

```java
// ❌ La referencia existe, el objeto no
Persona p = repo.buscar(id);   // devuelve null si no encuentra
p.getNombre();                 // 💥 NPE

// ✅ Devolvé Optional y forzá el manejo
Optional<Persona> p = repo.buscar(id);
p.map(Persona::getNombre).orElse("desconocido");

// ✅ Si comparás contra un literal, ponelo primero
if ("ACTIVO".equals(estado)) { ... }   // nunca falla, aunque estado sea null
if (estado.equals("ACTIVO")) { ... }   // 💥 si estado es null
```

> [!tip] Java 14+ te dice cuál referencia fue
> Los *helpful NullPointerExceptions* nombran la variable exacta. Si tu stacktrace no lo hace, activá `-XX:+ShowCodeDetailsInExceptionMessages`.

## Comparar con `==` en vez de `equals`

```java
String a = "hola";
String b = new String("hola");
a == b;        // false  ← compara referencias
a.equals(b);   // true   ← compara contenido

Integer x = 127, y = 127;   // x == y → true  (cache de -128..127)
Integer m = 128, n = 128;   // m == n → false ← el bug clásico
```

**Regla:** `==` solo para primitivos y para comparar contra `null`. Para todo lo demás, `equals`.

## Redefinir `equals` sin `hashCode`

```java
// ❌ Rompe HashMap, HashSet y distinct()
public class Cuenta {
    private String iban;
    @Override public boolean equals(Object o) { ... }
    // falta hashCode → dos objetos "iguales" caen en buckets distintos
}
```

**Contrato:** si `a.equals(b)` es `true`, entonces `a.hashCode() == b.hashCode()` **obligatoriamente**. Al revés no.
**Arreglo:** generá los dos juntos, o usá un `record`, que los implementa por vos.

## Mutar la colección mientras la recorrés

**Síntoma:** `ConcurrentModificationException` (sin hilos de por medio)

```java
// ❌
for (String s : lista) {
    if (s.startsWith("x")) lista.remove(s);   // 💥
}

// ✅ removeIf
lista.removeIf(s -> s.startsWith("x"));

// ✅ o Iterator explícito
var it = lista.iterator();
while (it.hasNext()) {
    if (it.next().startsWith("x")) it.remove();
}
```

## Confundir división entera

```java
int a = 5, b = 2;
double r = a / b;          // 2.0  ❌ divide como enteros y DESPUÉS convierte
double r = (double) a / b; // 2.5  ✅
```

## Arrays vs listas

```java
int[] arr = {1, 2, 3};
arr.length;          // propiedad, sin paréntesis
lista.size();        // método, con paréntesis
texto.length();      // método, con paréntesis

Arrays.asList(arr);              // ❌ con int[] devuelve List<int[]> de 1 elemento
Arrays.stream(arr).boxed().toList();  // ✅
```

---

# Nivel 2 — Diseño y API

## Devolver `null` en vez de vacío

```java
// ❌ obliga a que el llamador chequee null en cada uso
public List<Cuenta> buscar(...) { return encontradas.isEmpty() ? null : encontradas; }

// ✅
public List<Cuenta> buscar(...) { return encontradas; }   // lista vacía es una respuesta válida
```

## Usar excepciones para flujo normal

```java
// ❌ carísimo: construir el stacktrace domina el costo
try { return Integer.parseInt(s); } catch (NumberFormatException e) { return 0; }

// ✅ validá antes cuando el "error" es un caso esperado
if (s == null || !s.matches("-?\\d+")) return 0;
```

## Tragarse la excepción

```java
// ❌ el bug desaparece y nadie se entera
try { ... } catch (Exception e) { }
try { ... } catch (Exception e) { e.printStackTrace(); }   // tampoco: no va al log

// ✅
try { ... } catch (IOException e) {
    log.error("no se pudo leer {}", path, e);   // la causa viaja
    throw new AppException("error al procesar archivo", e);
}
```

## Perder la causa al reenvolver

```java
catch (SQLException e) {
    throw new AppException("error de base");      // ❌ se pierde el stacktrace original
    throw new AppException("error de base", e);   // ✅
}
```

## Clases con estado mutable compartido

```java
// ❌ un @Service es singleton: este campo es compartido por TODOS los requests
@Service
public class ReporteService {
    private List<Fila> acumulador = new ArrayList<>();   // 💥 race condition
}

// ✅ el estado vive en el método, no en el bean
public Reporte generar(...) {
    List<Fila> acumulador = new ArrayList<>();
    ...
}
```

---

# Nivel 3 — Dinero, fechas y datos

## Usar `double` para importes

```java
0.1 + 0.2 == 0.3          // false 💥
double total = 0.1 + 0.2; // 0.30000000000000004

// ✅ BigDecimal desde String, nunca desde double
new BigDecimal("0.1").add(new BigDecimal("0.2"));   // 0.3
new BigDecimal(0.1);                                 // ❌ ya viene con el error
```

## Comparar `BigDecimal` con `equals`

```java
new BigDecimal("1.0").equals(new BigDecimal("1.00"));    // false ← compara escala
new BigDecimal("1.0").compareTo(new BigDecimal("1.00")); // 0 ✅ compara valor
```

**Regla:** para importes, siempre `compareTo(...) == 0` y `signum()`.

## Dividir `BigDecimal` sin escala

```java
a.divide(b);                                    // 💥 ArithmeticException si es periódico
a.divide(b, 2, RoundingMode.HALF_UP);           // ✅ escala y modo de redondeo explícitos
```

## Fechas sin zona horaria

```java
LocalDateTime.now();     // ❌ "ahora" según la zona del servidor: cambia entre ambientes
Instant.now();           // ✅ instante absoluto en UTC
```

Persistí `Instant` (o `TIMESTAMP WITH TIME ZONE`) y convertí a zona local solo al mostrar.

## `SimpleDateFormat` compartido

`SimpleDateFormat` **no es thread-safe**: como campo `static` produce fechas corruptas bajo carga. Usá `DateTimeFormatter` (inmutable y seguro).

---

# Nivel 4 — Spring, JPA y producción

## Problema N+1

**Síntoma:** una consulta para la lista, y una consulta más *por cada* elemento.

```java
// ❌ 1 query de pedidos + N queries de cliente (lazy)
List<Pedido> pedidos = repo.findAll();
pedidos.forEach(p -> p.getCliente().getNombre());

// ✅ traé lo que vas a usar
@Query("SELECT p FROM Pedido p JOIN FETCH p.cliente")
List<Pedido> findAllConCliente();
```

Detectalo activando `spring.jpa.show-sql=true` y contando queries en un test.

## `@Transactional` que no hace nada

```java
@Service
public class PagoService {
    public void procesar() {
        registrar();      // ❌ llamada interna: el proxy de Spring NO intercepta
    }
    @Transactional public void registrar() { ... }
}
```

También falla si el método es `private`, `final` o `static`. **Arreglo:** poné `@Transactional` en el método de entrada, o separá en otro bean.

## `@Transactional` con checked exceptions

```java
@Transactional
public void procesar() throws IOException {
    ...
    throw new IOException();   // ❌ por defecto NO hace rollback
}

@Transactional(rollbackFor = Exception.class)   // ✅
```

Spring hace rollback automático solo ante `RuntimeException` y `Error`.

## Inyección por campo

```java
@Autowired private PagoRepository repo;   // ❌ no se puede testear sin Spring, oculta dependencias

// ✅ por constructor: dependencias explícitas e inmutables
private final PagoRepository repo;
public PagoService(PagoRepository repo) { this.repo = repo; }
```

## Entidades JPA como DTO de la API

Exponer la entidad filtra el modelo interno, arrastra relaciones lazy (`LazyInitializationException` al serializar) y acopla tu contrato público al esquema de base. Usá un `record` de respuesta.

## Falta de idempotencia

**Síntoma:** el cliente reintenta un `POST` por timeout y la transferencia se ejecuta dos veces.

```java
// ✅ clave de idempotencia: mismo request → mismo resultado, una sola ejecución
@PostMapping("/transferencias")
public Resp crear(@RequestHeader("Idempotency-Key") String key, @RequestBody Req req) { ... }
```

Es **el** error caro en banca. Se cubre en detalle en el módulo de banca y en el de sistemas distribuidos.

## Recursos sin cerrar

```java
// ❌ si falla, la conexión queda abierta y el pool se agota
var conn = ds.getConnection();
...
conn.close();

// ✅ try-with-resources cierra siempre
try (var conn = ds.getConnection()) { ... }
```

---

## 🎯 Checklist antes de un PR

- [ ] Ningún `catch` vacío ni con solo `printStackTrace`.
- [ ] Toda excepción reenvuelta conserva la causa.
- [ ] Ningún importe en `double` o `float`.
- [ ] `BigDecimal` comparado con `compareTo`, dividido con escala y `RoundingMode`.
- [ ] `equals` y `hashCode` redefinidos juntos (o `record`).
- [ ] Sin estado mutable en beans singleton.
- [ ] Sin entidades JPA en el contrato HTTP.
- [ ] Consultas revisadas por N+1.
- [ ] `@Transactional` en el método de entrada, no en uno llamado internamente.
- [ ] Recursos dentro de `try-with-resources`.

## 🔗 Relaciones
- **Base**: [[Java Fundamentos]], [[Java-cheatsheet]]
- **Profundizar**: [[09 - Excepciones]], [[04 - Optional]], [[Spring Data]]
- **Revisión de código**: [[01 - Code Review Senior]]

## 📌 Notas Rápidas
- Casi todos los errores de Nivel 1 los detecta el IDE o un linter; los de Nivel 3 y 4 solo aparecen bajo carga o con datos reales — por eso valen tests de integración.
- Cuando algo "funciona en local y falla en producción", sospechá primero de: zona horaria, concurrencia sobre estado compartido, y pool de conexiones agotado.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

