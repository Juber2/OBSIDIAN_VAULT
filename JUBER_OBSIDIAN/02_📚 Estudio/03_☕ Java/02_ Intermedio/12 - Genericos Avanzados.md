---
tags:
  - Java
  - Intermedio
Creado: 2026-09-10
Relacionado:
  - Java Intermedio
  - 01 - Generics
Palabras clave:
  - type erasure, wildcard, PECS, extends, super, covarianza, heap pollution, SafeVarargs
---

## 🧠 Concepto

[[01 - Generics]] cubre lo básico: `List<String>`, clases y métodos parametrizados. Esta nota cubre las dos cosas que hacen que los genéricos se vuelvan confusos: **el borrado de tipos** y **los comodines**.

## 🫥 Type erasure — el borrado de tipos

Los genéricos existen **solo en tiempo de compilación**. La JVM no los ve.

```java
// Lo que escribís                         Lo que existe en tiempo de ejecución
List<String> a = new ArrayList<>();        List a = new ArrayList();
List<Integer> b = new ArrayList<>();       List b = new ArrayList();

a.getClass() == b.getClass();              // true  ← son la MISMA clase
```

El compilador verifica los tipos, inserta las conversiones necesarias y después **borra** la información.

```java
// Lo que escribís
public <T> T primero(List<T> lista) { return lista.get(0); }

// Lo que queda tras el borrado
public Object primero(List lista) { return lista.get(0); }
// y en cada llamada, el compilador inserta el cast por vos
```

> [!tip] Por qué se hizo así
> Compatibilidad hacia atrás. Los genéricos llegaron en Java 5, y había que permitir que código nuevo y viejo convivieran en la misma JVM. El precio fue perder la información de tipos en ejecución.
> En C# los genéricos sí existen en ejecución (*reified*); en Java, no. Es la diferencia que explica todas las limitaciones que siguen.

### Las cinco cosas que no podés hacer

```java
// 1 · Crear un array de un tipo genérico
T[] array = new T[10];                       // ❌ no compila
T[] array = (T[]) new Object[10];            // ⚠️ funciona, con warning

// 2 · Preguntar por el tipo concreto
if (lista instanceof List<String>) { }       // ❌ no compila
if (lista instanceof List<?>) { }            // ✅ lo único permitido

// 3 · Campos static del tipo parametrizado
public class Caja<T> {
    private static T valor;                  // ❌ no compila: T no existe por clase
}

// 4 · Sobrecargar por el tipo genérico
void procesar(List<String> l) { }
void procesar(List<Integer> l) { }           // ❌ tras el borrado son el mismo método

// 5 · Capturar una excepción genérica
try { } catch (T e) { }                      // ❌ no compila
```

### Cómo se sortea: el token de clase

```java
public class Repositorio<T> {
    private final Class<T> tipo;             // ← se guarda el tipo explícitamente

    public Repositorio(Class<T> tipo) { this.tipo = tipo; }

    @SuppressWarnings("unchecked")
    public T[] crearArray(int tamano) {
        return (T[]) Array.newInstance(tipo, tamano);    // ahora sí
    }

    public boolean esDelTipo(Object o) { return tipo.isInstance(o); }
}

var repo = new Repositorio<>(Cuenta.class);
```

Es el patrón que usan `Class<T>`, Jackson (`readValue(json, Cuenta.class)`) y Spring por todos lados.

## 🃏 Comodines

El problema que resuelven:

```java
List<Object> objetos = new ArrayList<String>();     // ❌ NO compila
```

> [!warning] Los genéricos son invariantes
> `List<String>` **no** es subtipo de `List<Object>`, aunque `String` sí lo sea de `Object`.
> Y tiene que ser así:
> ```java
> List<String> textos = new ArrayList<>();
> List<Object> objetos = textos;              // si esto compilara...
> objetos.add(42);                            // ...meterías un Integer
> String s = textos.get(0);                   // 💥 ClassCastException
> ```
> Los arrays sí son covariantes, y por eso `Object[] a = new String[1]; a[0] = 42;` compila y falla en ejecución con `ArrayStoreException`. Los genéricos evitaron ese error de diseño.

### `? extends` — solo lectura

```java
public double sumar(List<? extends Number> numeros) {
    double total = 0;
    for (Number n : numeros) total += n.doubleValue();   // ✅ LEER está bien
    return total;
}

sumar(List.of(1, 2, 3));            // List<Integer>  ✅
sumar(List.of(1.5, 2.5));           // List<Double>   ✅
```

```java
public void agregar(List<? extends Number> lista) {
    lista.add(42);                   // ❌ no compila
}
```

> [!tip] Por qué no se puede escribir
> `List<? extends Number>` significa *"una lista de algún subtipo de Number, no sé cuál"*. Si fuera `List<Double>`, agregar un `Integer` la corrompería. Como el compilador no sabe cuál es, prohíbe escribir.
> Lo único que se puede agregar es `null`, que es válido para cualquier tipo.

### `? super` — solo escritura

```java
public void llenar(List<? super Integer> lista) {
    lista.add(1);                    // ✅ ESCRIBIR está bien
    lista.add(2);
}

llenar(new ArrayList<Integer>());    // ✅
llenar(new ArrayList<Number>());     // ✅
llenar(new ArrayList<Object>());     // ✅
```

```java
public void leer(List<? super Integer> lista) {
    Integer x = lista.get(0);        // ❌ no compila
    Object  o = lista.get(0);        // ✅ lo único seguro
}
```

Es *"una lista de algún supertipo de Integer"*: podés meter `Integer` con seguridad, pero al leer lo único garantizado es `Object`.

## 🐷 PECS

La regla mnemotécnica para decidir cuál usar:

> **P**roducer **E**xtends, **C**onsumer **S**uper

```
¿El parámetro PRODUCE datos que vas a leer?     → ? extends
¿El parámetro CONSUME datos que vas a escribir? → ? super
¿Las dos cosas?                                  → sin comodín, tipo exacto
```

```java
// El ejemplo canónico, de la propia biblioteca estándar
public static <T> void copy(List<? super T> destino, List<? extends T> origen) {
    //                            ↑ consume            ↑ produce
    for (int i = 0; i < origen.size(); i++) {
        destino.set(i, origen.get(i));
    }
}
```

```java
// En Stream
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
//                          ↑ consume T      ↑ produce R
```

> [!tip] PECS se lee desde el punto de vista del parámetro, no del método
> La lista de origen **produce** elementos para que vos los leas → `extends`. La lista destino **consume** los elementos que vos le das → `super`.
> Es la confusión inicial más común: pensarlo desde lo que hace el método en vez de lo que hace el parámetro.

### Comodín sin límites

```java
public int contar(List<?> lista) {           // "una lista de algo"
    return lista.size();                      // solo operaciones que no dependen del tipo
}
```

Sirve cuando no te importa el tipo. `List<?>` es distinto de `List<Object>`: a la primera no le podés agregar nada, a la segunda sí.

## 🎯 Límites en los parámetros de tipo

```java
// T tiene que ser Comparable consigo mismo
public static <T extends Comparable<T>> T maximo(List<T> lista) {
    T max = lista.get(0);
    for (T e : lista) if (e.compareTo(max) > 0) max = e;
    return max;
}

// Varios límites: primero la clase, después las interfaces
public <T extends Number & Comparable<T> & Serializable> void procesar(T valor) { }
```

```java
// Límite recursivo: el patrón que usan los enums y los builders
public abstract class Constructor<T extends Constructor<T>> {
    @SuppressWarnings("unchecked")
    public T conNombre(String n) { this.nombre = n; return (T) this; }
}
public class ConstructorCuenta extends Constructor<ConstructorCuenta> { }
// Permite encadenar devolviendo el tipo HIJO, no el padre
```

## ☠️ Contaminación del heap

```java
@SafeVarargs
public static <T> List<T> deElementos(T... elementos) {
    return List.of(elementos);
}
```

```java
// Sin @SafeVarargs, el compilador advierte. Y a veces tiene razón:
public static <T> T[] peligroso(T... elementos) {
    return elementos;                        // devuelve un array de tipo borrado
}

String[] textos = peligroso("a", "b");       // 💥 ClassCastException en ejecución
```

> [!warning] `@SafeVarargs` es una promesa tuya, no una verificación
> El compilador no comprueba nada: solo silencia la advertencia. Solo ponela si el método **realmente** no escribe en el array ni lo expone hacia afuera.
> Un varargs genérico crea internamente un `Object[]`, y si lo devolvés como `T[]`, el cast implícito en el llamador explota.

## 🧭 Guía rápida

| Situación | Qué usar |
|---|---|
| El método solo lee de la colección | `List<? extends T>` |
| El método solo escribe en la colección | `List<? super T>` |
| Lee y escribe | `List<T>` |
| No importa el tipo | `List<?>` |
| Necesitás el tipo en ejecución | pasar `Class<T>` |
| El tipo debe tener ciertas capacidades | `<T extends Comparable<T>>` |

> [!tip] En los tipos de retorno, no uses comodines
> ```java
> public List<? extends Number> obtener() { ... }   // ❌ obliga al llamador a lidiar con el comodín
> public List<Number> obtener() { ... }             // ✅
> ```
> Los comodines van en los **parámetros**, para aceptar más tipos. En el retorno solo trasladan la complejidad a quien te llama.

## 🧪 Para probar

```java
@Test
void el_borrado_hace_que_dos_listas_distintas_sean_la_misma_clase() {
    assertSame(new ArrayList<String>().getClass(), new ArrayList<Integer>().getClass());
}

@Test
void extends_permite_leer_de_cualquier_subtipo() {
    assertEquals(6.0, sumar(List.of(1, 2, 3)));
    assertEquals(6.0, sumar(List.of(1.0, 2.0, 3.0)));
}

@Test
void super_permite_escribir_en_cualquier_supertipo() {
    List<Number> numeros = new ArrayList<>();
    llenar(numeros);
    assertEquals(2, numeros.size());
}

@Test
void el_token_de_clase_recupera_el_tipo_en_ejecucion() {
    var repo = new Repositorio<>(Cuenta.class);
    assertTrue(repo.esDelTipo(new Cuenta("ACC-001")));
}
```

## ⚠️ Errores Comunes

> [!warning] Los que confunden al empezar
> - **Esperar que `List<String>` sea un `List<Object>`**: los genéricos son invariantes.
> - **Intentar escribir en un `? extends`** o **leer tipado de un `? super`**.
> - **Comodines en el tipo de retorno**: complican a quien llama.
> - **`instanceof List<String>`**: no existe en ejecución.
> - **Sobrecargar métodos por el tipo genérico**: tras el borrado son el mismo.
> - **`@SafeVarargs` sin verificar** que el método sea realmente seguro.
> - **Usar tipos crudos (`List` sin parámetro)** para "evitar problemas": desactiva toda la verificación.

```java
List crudo = new ArrayList();      // ❌ tipo crudo: el compilador deja de ayudarte
crudo.add("texto");
crudo.add(42);
String s = (String) crudo.get(1);  // 💥 ClassCastException
```

## 📌 Notas Rápidas
- Todo lo raro de los genéricos en Java se explica con una frase: **existen en compilación y no en ejecución**. El borrado es la causa de las cinco limitaciones y del patrón del token de clase.
- **PECS** es la regla que más rinde memorizar: *Producer Extends, Consumer Super*. Aparece en toda la biblioteca estándar y en cualquier API genérica bien diseñada.
- Si te encontrás peleando con comodines, muchas veces la señal es que el método hace demasiadas cosas. Separar lectura y escritura suele simplificar la firma.
- Los tipos crudos existen solo por compatibilidad con código anterior a Java 5. En código nuevo son siempre un error.

## 🔗 Relaciones
- **Módulo**: [[Java Intermedio]]
- **Base**: [[01 - Generics]]
- **Se conecta con**: [[11 - Comparable y Comparator]], [[03 - Streams API]], [[05 - Reflection]], [[09 - Records y Sealed Classes]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
