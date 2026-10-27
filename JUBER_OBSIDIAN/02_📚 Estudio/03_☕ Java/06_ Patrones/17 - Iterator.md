---
tags:
  - Java
  - Patrones
  - Comportamiento
Creado: 2026-09-10
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - iterator, Iterable, for-each, recorrido, paginacion, ConcurrentModificationException
---

## 🧠 Concepto

Permite recorrer los elementos de una colección **sin exponer su estructura interna**. Quien recorre no necesita saber si adentro hay un array, una lista enlazada o un árbol.

**Cuándo usarlo:** casi nunca lo escribís a mano — Java ya lo tiene integrado. Importa entenderlo porque **es el patrón que hace funcionar el `for-each`**, y porque a veces sí necesitás uno propio.

## Está integrado en el lenguaje

```java
for (Cuenta c : cuentas) { ... }
```

El compilador lo traduce a:

```java
Iterator<Cuenta> it = cuentas.iterator();
while (it.hasNext()) {
    Cuenta c = it.next();
    ...
}
```

Cualquier clase que implemente `Iterable<T>` funciona con `for-each`. Esa es toda la magia.

## Implementación propia

```java
public class Libro implements Iterable<Asiento> {

    private final List<Asiento> asientos = new ArrayList<>();

    @Override
    public Iterator<Asiento> iterator() {
        return new Iterator<>() {                  // clase anónima
            private int indice = 0;

            @Override public boolean hasNext() { return indice < asientos.size(); }
            @Override public Asiento next() {
                if (!hasNext()) throw new NoSuchElementException();
                return asientos.get(indice++);
            }
        };
    }
}
```

```java
Libro libro = new Libro();
for (Asiento a : libro) {                          // ✅ funciona
    System.out.println(a.motivo());
}
```

> [!tip] Lo más simple: delegar
> Si internamente ya tenés una colección, no escribas el iterador a mano:
> ```java
> @Override public Iterator<Asiento> iterator() { return asientos.iterator(); }
> ```
> Pero **cuidado**: ese iterador expone `remove()`, así que quien recorra puede modificar tu colección interna. Para impedirlo:
> ```java
> @Override public Iterator<Asiento> iterator() {
>     return Collections.unmodifiableList(asientos).iterator();
> }
> ```

## Un iterador que no carga todo en memoria

Acá está el uso real que justifica escribir uno propio: recorrer millones de registros sin materializarlos.

```java
public class ApuntesPaginados implements Iterable<Apunte> {

    private final ApunteRepository repo;
    private final CuentaId cuenta;
    private static final int TAMANO = 500;

    @Override
    public Iterator<Apunte> iterator() {
        return new Iterator<>() {
            private List<Apunte> lote = List.of();
            private int indice = 0;
            private Long cursor = null;
            private boolean agotado = false;

            @Override
            public boolean hasNext() {
                if (indice < lote.size()) return true;
                if (agotado) return false;
                cargarSiguienteLote();
                return indice < lote.size();
            }

            @Override
            public Apunte next() {
                if (!hasNext()) throw new NoSuchElementException();
                Apunte a = lote.get(indice++);
                cursor = a.id();
                return a;
            }

            private void cargarSiguienteLote() {
                lote = repo.siguientePagina(cuenta, cursor, TAMANO);
                indice = 0;
                if (lote.size() < TAMANO) agotado = true;
            }
        };
    }
}
```

```java
// El que recorre no se entera de que hay paginación por detrás
for (Apunte a : new ApuntesPaginados(repo, cuenta)) {
    procesar(a);
}
```

> [!tip] Es el patrón detrás de los lectores de Spring Batch
> `JdbcPagingItemReader` hace exactamente esto: expone una interfaz de "dame el siguiente" y por debajo va trayendo páginas. Por eso un batch puede recorrer 8 millones de cuentas con memoria constante — ver [[10 - Cierre de Dia y Batch]].

## 💥 `ConcurrentModificationException`

```java
for (String s : lista) {
    if (s.startsWith("x")) lista.remove(s);        // 💥
}
```

El iterador lleva un contador de modificaciones (`modCount`) y verifica en cada `next()` que la colección no haya cambiado por fuera.

```java
// ✅ Con el propio iterador
Iterator<String> it = lista.iterator();
while (it.hasNext()) {
    if (it.next().startsWith("x")) it.remove();     // remove del ITERADOR
}

// ✅ Más corto
lista.removeIf(s -> s.startsWith("x"));
```

> [!warning] El nombre confunde: no tiene que ver con hilos
> Ocurre con un solo hilo. Es una **detección de bug**, no un mecanismo de sincronización — y ni siquiera está garantizada: la documentación dice que es de "mejor esfuerzo".

## Iterador vs Stream

```java
for (Apunte a : apuntes) { ... }              // Iterator: control explícito, se puede cortar
apuntes.stream().forEach(this::procesar);     // Stream: declarativo, componible
```

| | Iterator / for-each | Stream |
|---|---|---|
| Cortar en el medio | `break` | `takeWhile`, `findFirst` |
| Modificar mientras recorre | `it.remove()` | ❌ |
| Encadenar operaciones | manual | `filter`, `map`, `collect` |
| Paralelizar | manual | `parallelStream()` |
| Reutilizable | sí, pedís otro iterador | ❌ un solo uso |

```java
// De Iterable a Stream cuando lo necesitás
StreamSupport.stream(miIterable.spliterator(), false)
    .filter(...)
    .toList();
```

## 📌 Notas Rápidas
- Es el patrón que **menos vas a implementar y más vas a usar**: cada `for-each` que escribís lo está usando.
- La razón real para escribir uno propio es la carga perezosa: recorrer un origen enorme (base, archivo, API paginada) sin traerlo todo a memoria.
- `Iterable` es lo que habilita el `for-each`; `Iterator` es el que lleva la posición. Una colección puede dar muchos iteradores independientes.
- Si exponés `iterator()` delegando en tu colección interna, envolvela en `unmodifiableList` o alguien la va a modificar desde afuera — mismo criterio que la copia defensiva de [[10 - equals hashCode e Inmutabilidad]].
- Se relaciona con [[11 - Internals de Colecciones]], donde está el detalle del `modCount`.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
