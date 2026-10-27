---
tags:
  - Java
  - Intermedio
Creado: 2026-09-10
Relacionado:
  - Java Intermedio
  - 08 - Arrays y Colecciones
Palabras clave:
  - Comparable, Comparator, compareTo, orden natural, thenComparing, TreeSet, nullsFirst
---

## 🧠 Concepto

Java no sabe cómo ordenar tus objetos: se lo tenés que decir. Hay dos formas, y la diferencia es **quién define el criterio**.

| | `Comparable` | `Comparator` |
|---|---|---|
| Dónde vive el criterio | **dentro** de la clase | **fuera**, en un objeto aparte |
| Cuántos puede haber | uno solo (orden natural) | tantos como quieras |
| Método | `compareTo(otro)` | `compare(a, b)` |
| Requiere modificar la clase | ✅ | ❌ |
| Sirve para clases ajenas | ❌ | ✅ |

## 1️⃣ `Comparable` — el orden natural

```java
public record Cuenta(String numero, Dinero saldo) implements Comparable<Cuenta> {

    @Override
    public int compareTo(Cuenta otra) {
        return this.numero.compareTo(otra.numero);      // orden natural: por número
    }
}
```

```java
List<Cuenta> cuentas = new ArrayList<>(...);
Collections.sort(cuentas);          // usa compareTo
cuentas.sort(null);                 // idem
new TreeSet<>(cuentas);             // idem
```

**El contrato del valor devuelto:**

```
negativo  →  this va ANTES que otro
cero      →  son equivalentes en orden
positivo  →  this va DESPUÉS que otro
```

> [!tip] Implementalo cuando exista un orden obvio y único
> Un número de cuenta, una fecha, un importe. Si tenés que preguntarte *"¿ordenado por qué?"*, entonces no hay orden natural y corresponde `Comparator`.

## 2️⃣ `Comparator` — criterios externos

```java
Comparator<Cuenta> porSaldo = (a, b) -> a.saldo().compareTo(b.saldo());

// Mucho mejor con los métodos de fábrica
Comparator<Cuenta> porSaldo = Comparator.comparing(Cuenta::saldo);
```

```java
cuentas.sort(porSaldo);
cuentas.stream().sorted(porSaldo).toList();
new TreeSet<>(porSaldo);
```

### Composición

```java
Comparator<Cliente> orden = Comparator
    .comparing(Cliente::apellido)                       // primero por apellido
    .thenComparing(Cliente::nombre)                     // empate → por nombre
    .thenComparing(Cliente::fechaAlta, reverseOrder())  // empate → más reciente primero
    .reversed();                                        // invierte TODO lo anterior
```

> [!warning] `reversed()` invierte la cadena completa, no el último criterio
> Es un error frecuente. Si querés invertir solo un criterio, pasalo como segundo argumento de `comparing`:
> ```java
> Comparator.comparing(Cliente::apellido)
>           .thenComparing(Cliente::fechaAlta, Comparator.reverseOrder());
> ```

### Variantes que evitan boxing

```java
Comparator.comparing(Cuenta::cantidadMovimientos);       // devuelve Integer → boxing
Comparator.comparingInt(Cuenta::cantidadMovimientos);    // ✅ int, sin boxing
Comparator.comparingLong(...);   Comparator.comparingDouble(...);
```

En listas grandes la diferencia se nota — ver [[11 - Wrappers y Autoboxing]].

### Nulos

```java
// ❌ NullPointerException si algún apellido es null
Comparator.comparing(Cliente::apellido);

// ✅ Los null van primero (o último)
Comparator.comparing(Cliente::apellido, Comparator.nullsFirst(Comparator.naturalOrder()));
Comparator.nullsLast(Comparator.comparing(Cliente::apellido));   // el OBJETO puede ser null
```

> [!tip] Hay dos nulos posibles y se manejan distinto
> `nullsLast(comparing(...))` protege contra **el objeto** nulo. `comparing(f, nullsLast(...))` protege contra **el campo** nulo. Confundirlos es la causa del NPE que sigue apareciendo después de "arreglarlo".

## 💥 El bug de la resta

```java
// ❌ El clásico
Comparator<Cuenta> porId = (a, b) -> a.id() - b.id();
```

Parece correcto y **desborda**:

```java
int a = Integer.MAX_VALUE;      //  2147483647
int b = -1;
a - b;                           // -2147483648  ← negativo: dice que a va antes que b
```

```java
// ✅
Comparator<Cuenta> porId = Comparator.comparingInt(Cuenta::id);
// o
(a, b) -> Integer.compare(a.id(), b.id());
```

> [!danger] Es un bug que sobrevive a todas las pruebas
> Con ids chicos la resta funciona perfecto. Falla solo cuando los valores se acercan a los extremos del rango — y cuando falla, el resultado es una lista mal ordenada sin ninguna excepción que lo delate.

## 📜 El contrato

```
1 · Antisimetría:   sgn(compare(x, y)) == -sgn(compare(y, x))
2 · Transitividad:  si compare(x,y) > 0 y compare(y,z) > 0 → compare(x,z) > 0
3 · Consistencia:   si compare(x,y) == 0 → sgn(compare(x,z)) == sgn(compare(y,z))
4 · Recomendado:    compare(x,y) == 0  ⟺  x.equals(y)
```

> [!danger] Romper el contrato lanza una excepción confusa
> ```
> java.lang.IllegalArgumentException: Comparison method violates its general contract!
> ```
> El algoritmo de ordenamiento de Java (TimSort) **detecta** comparadores inconsistentes y aborta. El mensaje no dice cuál es el problema ni en qué elemento, así que rastrearlo es tedioso.
> Causas habituales: la resta que desborda, un comparador que usa un campo mutable que cambió durante el ordenamiento, o ramas que no cubren todos los casos.

### La regla 4 y por qué importa

```java
public record Producto(String sku, String nombre) implements Comparable<Producto> {
    @Override public int compareTo(Producto otro) {
        return nombre.compareTo(otro.nombre);      // ordena por NOMBRE
    }
    // equals (de record) compara sku Y nombre
}
```

```java
var a = new Producto("SKU-1", "Café");
var b = new Producto("SKU-2", "Café");      // mismo nombre, distinto sku

a.equals(b);            // false  ← son productos distintos
a.compareTo(b);         // 0      ← "equivalentes" según el orden

Set<Producto> hash = new HashSet<>(List.of(a, b));    // tamaño 2  ✅
Set<Producto> tree = new TreeSet<>(List.of(a, b));    // tamaño 1  💥 perdiste uno
```

> [!warning] `TreeSet` y `TreeMap` usan `compareTo`, no `equals`
> Si el orden dice "iguales" pero `equals` dice "distintos", el `TreeSet` **descarta elementos silenciosamente**. No hay error: simplemente falta un producto.
> Por eso el contrato recomienda que sean consistentes. Si no podés hacerlos consistentes, desempatá con un campo único:
> ```java
> return Comparator.comparing(Producto::nombre)
>                  .thenComparing(Producto::sku)      // desempate por identidad
>                  .compare(this, otro);
> ```

## 🔤 Casos particulares

```java
// Texto con acentos y ñ: el orden natural de String es por código Unicode
"ñ".compareTo("z");                       // positivo → "ñ" va DESPUÉS de "z" ❌

Collator collator = Collator.getInstance(new Locale("es", "PE"));
lista.sort(collator::compare);            // ✅ orden alfabético español

// Ignorar mayúsculas
String.CASE_INSENSITIVE_ORDER;
Comparator.comparing(Cliente::nombre, String.CASE_INSENSITIVE_ORDER);
```

```java
// Ordenar por un criterio arbitrario definido por vos
List<String> prioridad = List.of("CRITICA", "ALTA", "MEDIA", "BAJA");
Comparator<Incidente> porPrioridad =
    Comparator.comparingInt(i -> prioridad.indexOf(i.severidad()));
```

```java
// Enums: el orden natural es el de DECLARACIÓN
public enum Severidad { CRITICA, ALTA, MEDIA, BAJA }
incidentes.sort(Comparator.comparing(Incidente::severidad));   // ya sale bien
```

> [!tip] Aprovechá el orden de declaración de los enums
> `Enum` implementa `Comparable` usando `ordinal()`. Si declarás las constantes en el orden que necesitás, no hace falta ningún comparador.
> El riesgo: reordenar las constantes cambia el orden en todo el sistema. Si el orden es parte del negocio, mejor un campo explícito.

## 🔒 Estabilidad

```java
Collections.sort(lista);        // ESTABLE: los empates conservan el orden previo
Arrays.sort(primitivos);        // NO estable (quicksort para primitivos)
```

Un ordenamiento estable permite **encadenar**: ordenar por nombre y después por apellido deja los del mismo apellido ordenados por nombre. Con uno inestable eso no funciona.

## 🧪 Para probar

```java
@Test
void la_resta_desborda_y_el_orden_sale_mal() {
    Comparator<Integer> malo = (a, b) -> a - b;
    assertTrue(malo.compare(Integer.MAX_VALUE, -1) < 0);   // ¡dice que MAX es menor!

    Comparator<Integer> bueno = Integer::compare;
    assertTrue(bueno.compare(Integer.MAX_VALUE, -1) > 0);  // ✅
}

@Test
void un_TreeSet_descarta_elementos_si_compareTo_no_coincide_con_equals() {
    var set = new TreeSet<>(List.of(
        new Producto("SKU-1", "Café"),
        new Producto("SKU-2", "Café")));
    assertEquals(1, set.size());        // documenta el comportamiento
}

@Test
void el_comparador_es_antisimetrico() {
    assertEquals(-Integer.signum(cmp.compare(b, a)), Integer.signum(cmp.compare(a, b)));
}

@Test
void nullsLast_no_lanza_NPE() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que rompen el orden en silencio
> - **`a - b` para comparar**: desborda con valores extremos.
> - **`compareTo` inconsistente con `equals`**: `TreeSet` descarta elementos.
> - **`reversed()`** creyendo que invierte solo el último criterio.
> - **Confundir `nullsLast(comparing(f))` con `comparing(f, nullsLast(...))`.**
> - **Comparar por un campo mutable** que cambia durante el ordenamiento: `IllegalArgumentException`.
> - **`comparing` en vez de `comparingInt`** en listas grandes: boxing innecesario.
> - **Orden natural de `String`** para texto en español: la `ñ` queda después de la `z`.
> - **`Comparable` con más de un criterio posible**: si hay dos formas válidas de ordenar, ninguna es "natural".

## 📌 Notas Rápidas
- Regla de decisión: **`Comparable` si el orden es una propiedad del tipo; `Comparator` si es una decisión de quien lo usa.**
- El mensaje *"Comparison method violates its general contract"* siempre significa que tu comparador es inconsistente. Las tres causas habituales son la resta que desborda, un campo mutable, o ramas incompletas.
- `Comparator.comparing(...).thenComparing(...)` reemplazó casi por completo a escribir `compare` a mano. Es más legible y no tiene el bug de la resta.
- La inconsistencia entre `compareTo` y `equals` es sutil y se paga en `TreeSet`/`TreeMap` — conecta con [[10 - equals hashCode e Inmutabilidad]].

## 🔗 Relaciones
- **Módulo**: [[Java Intermedio]]
- **Base**: [[08 - Arrays y Colecciones]], [[02 - Lambdas y Functional Interfaces]]
- **Se conecta con**: [[10 - equals hashCode e Inmutabilidad]], [[11 - Internals de Colecciones]], [[03 - Streams API]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
