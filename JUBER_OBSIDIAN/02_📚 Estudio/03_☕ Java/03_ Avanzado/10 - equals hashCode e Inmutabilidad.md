---
tags:
  - Java
  - Avanzado
  - POO
Creado: 2026-09-08
Relacionado:
  - Java Avanzado
  - 09 - Records y Sealed Classes
Palabras clave:
  - equals, hashCode, contrato, inmutabilidad, record, simetria, copia defensiva
---

## 🧠 Concepto

`equals` y `hashCode` parecen dos métodos que el IDE genera solos. En realidad son **un contrato**, y romperlo produce bugs que no lanzan excepción: un `HashMap` que no encuentra lo que acabás de guardar, un `distinct()` que deja duplicados, un `Set` con dos elementos iguales.

## 📜 El contrato de `equals`

| Propiedad | Significa |
|---|---|
| **Reflexiva** | `x.equals(x)` es `true` |
| **Simétrica** | si `x.equals(y)`, entonces `y.equals(x)` |
| **Transitiva** | si `x.equals(y)` e `y.equals(z)`, entonces `x.equals(z)` |
| **Consistente** | invocaciones repetidas dan lo mismo si el objeto no cambió |
| **No nula** | `x.equals(null)` es `false`, nunca lanza excepción |

## 📜 El contrato de `hashCode`

```
Si a.equals(b) → a.hashCode() == b.hashCode()      OBLIGATORIO
Si a.hashCode() == b.hashCode() → a.equals(b)?     NO necesariamente (colisión)
```

> [!danger] Redefinir `equals` sin `hashCode` rompe todas las estructuras basadas en hash
> ```java
> var mapa = new HashMap<Cuenta, Saldo>();
> mapa.put(new Cuenta("ACC-001"), saldo);
> mapa.get(new Cuenta("ACC-001"));      // → null 💥
> ```
> El `HashMap` calcula el bucket con `hashCode`. Si dos objetos "iguales" tienen hashes distintos, caen en buckets distintos y **nunca se comparan**. No hay error, no hay excepción: simplemente no lo encuentra.

## ✅ Implementación correcta

```java
public final class Cuenta {

    private final String numero;
    private final Moneda moneda;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                       // atajo: misma referencia
        if (!(o instanceof Cuenta otra)) return false;    // cubre null y tipo
        return numero.equals(otra.numero) && moneda == otra.moneda;
    }

    @Override
    public int hashCode() {
        return Objects.hash(numero, moneda);              // MISMOS campos que equals
    }
}
```

> [!tip] La regla de oro
> **Los campos que participan en `equals` son exactamente los que participan en `hashCode`.** Ni uno más, ni uno menos. Si el IDE los genera juntos, ya está; si los editás a mano, editá los dos.

## ⚠️ Las cuatro trampas

### 1 · Campos mutables en `hashCode`

```java
// ❌ El objeto "se pierde" dentro del Set
var cuenta = new Cuenta("ACC-001", Estado.ACTIVA);
var set = new HashSet<Cuenta>();
set.add(cuenta);

cuenta.setEstado(Estado.BLOQUEADA);      // cambia el hashCode
set.contains(cuenta);                     // → false 💥
```

El objeto está en el bucket calculado con el hash **viejo**; `contains` lo busca en el bucket del hash **nuevo**. Queda inaccesible, pero ocupando memoria.

> [!warning] Solo campos inmutables en `hashCode`
> Y la forma más simple de garantizarlo es que el objeto entero sea inmutable.

### 2 · Simetría rota por herencia

```java
public class Punto {
    protected final int x, y;
    @Override public boolean equals(Object o) {
        return o instanceof Punto p && p.x == x && p.y == y;
    }
}

public class PuntoColoreado extends Punto {
    private final Color color;
    @Override public boolean equals(Object o) {
        return o instanceof PuntoColoreado p && super.equals(p) && p.color == color;
    }
}
```

```java
var p  = new Punto(1, 2);
var pc = new PuntoColoreado(1, 2, ROJO);

p.equals(pc);    // true  ← Punto solo compara x, y
pc.equals(p);    // false ← PuntoColoreado exige color
                 // 💥 SIMETRÍA ROTA
```

> [!danger] No hay forma de extender una clase concreta agregando estado y preservar el contrato
> Es un resultado conocido (lo desarrolla *Effective Java*): con `instanceof` se rompe la simetría; con `getClass()` se rompe el principio de sustitución de Liskov.
> **La salida es composición, no herencia:**
> ```java
> public final class PuntoColoreado {
>     private final Punto punto;      // composición
>     private final Color color;
> }
> ```

### 3 · `instanceof` vs `getClass()`

| | `instanceof` | `getClass()` |
|---|---|---|
| Con subclases | rompe simetría | rompe sustitución |
| Con proxies (Hibernate, Spring) | ✅ funciona | ❌ **falla** |

> [!warning] Con Hibernate, `getClass()` rompe
> Una entidad cargada de forma perezosa es un **proxy**, no la clase real. `getClass()` devuelve `Cuenta$HibernateProxy$abc123`, así que comparar una entidad cargada con una nueva da `false` aunque tengan el mismo id.
> Con `instanceof` (o `Hibernate.unproxy()`) funciona.

### 4 · `equals` en entidades JPA

```java
// ❌ Usar el id generado: antes de persistir es null
@Override public boolean equals(Object o) {
    return o instanceof CuentaEntity c && Objects.equals(id, c.id);
}
// dos entidades nuevas sin persistir tienen id null → se consideran iguales 💥
```

```java
// ✅ Clave de negocio, no el id técnico
@Override public boolean equals(Object o) {
    return o instanceof CuentaEntity c && Objects.equals(numero, c.numero);
}

@Override public int hashCode() {
    return Objects.hash(numero);      // estable durante todo el ciclo de vida
}
```

> [!tip] El `hashCode` de una entidad JPA debe ser estable antes y después de persistir
> Si depende del id generado, cambia al hacer `flush` — y si la entidad estaba en un `Set`, se pierde. Usar una clave de negocio inmutable lo evita.
> Si no hay clave de negocio, una opción válida es `hashCode()` constante (por ejemplo `31`): degrada el rendimiento del `Set` pero mantiene la corrección.

## 🎁 `record`: el contrato gratis

```java
public record Cuenta(String numero, Moneda moneda) { }
```

El compilador genera `equals`, `hashCode` y `toString` sobre **todos** los componentes, correctamente y sin posibilidad de desincronizarlos.

> [!danger] La trampa de `record` con `BigDecimal`
> `record` genera `equals` que compara campo a campo con `equals`. Y `BigDecimal.equals` compara **escala**:
> ```java
> record Precio(BigDecimal monto) { }
> new Precio(new BigDecimal("1.0")).equals(new Precio(new BigDecimal("1.00")));  // false 💥
> ```
> La solución es normalizar en el constructor compacto:
> ```java
> public record Dinero(BigDecimal monto, Moneda moneda) {
>     public Dinero {
>         monto = monto.setScale(moneda.decimales(), RoundingMode.HALF_UP);
>     }
> }
> ```
> Es exactamente lo que hace el value object de [[01 - Dinero en Software]].

> [!warning] `record` con arrays no funciona como esperás
> `Object.equals` de un array compara referencias, no contenido. Un `record` con un `byte[]` casi nunca hace lo que querés: usá una `List` inmutable o redefiní `equals` a mano.

## 🧊 Inmutabilidad

Un objeto inmutable resuelve de un saque los problemas de `hashCode` inestable, publicación insegura y estado compartido.

```java
public final class Periodo {                              // 1 · final: sin subclases

    private final LocalDate inicio;                        // 2 · campos final
    private final List<Feriado> feriados;

    public Periodo(LocalDate inicio, List<Feriado> feriados) {
        this.inicio = inicio;
        this.feriados = List.copyOf(feriados);             // 3 · copia defensiva al ENTRAR
    }

    public List<Feriado> feriados() {
        return feriados;                                   // 4 · ya es inmutable
    }

    public Periodo conInicio(LocalDate nuevo) {            // 5 · "modificar" = crear otro
        return new Periodo(nuevo, feriados);
    }
}
```

> [!danger] La copia defensiva no es opcional
> ```java
> var lista = new ArrayList<>(List.of(feriado1));
> var periodo = new Periodo(inicio, lista);
> lista.add(feriado2);        // ❌ sin copia, el "inmutable" acaba de cambiar
> ```
> Sin copia al entrar, quien construyó el objeto conserva una referencia mutable a su estado interno. Lo mismo al salir: devolver la colección interna permite modificarla desde afuera.

### Qué gana la inmutabilidad

| Beneficio | Por qué |
|---|---|
| Segura para hilos | no hay escrituras que sincronizar |
| `hashCode` estable | se puede cachear; funciona en `Set` y `Map` |
| Sin copias defensivas al compartir | no hay nada que proteger |
| Razonamiento simple | el valor no cambia bajo tus pies |
| Publicación segura | los campos `final` la garantizan ([[09 - Modelo de Memoria de Java]]) |

> [!tip] Cachear el `hashCode` de un inmutable
> ```java
> private int hash;      // no volatile: la carrera es benigna, el valor es el mismo
> @Override public int hashCode() {
>     int h = hash;
>     if (h == 0) hash = h = Objects.hash(campo1, campo2);
>     return h;
> }
> ```
> Es lo que hace `String` internamente. Solo es correcto porque el objeto es inmutable.

## 📊 Calidad del `hashCode`

```java
// ❌ Todos los objetos al mismo bucket: el HashMap se degrada a lista enlazada
@Override public int hashCode() { return 1; }

// ⚠️ Mala distribución si un campo domina
@Override public int hashCode() { return numero.length(); }

// ✅ Distribución razonable
@Override public int hashCode() { return Objects.hash(numero, moneda); }
```

| Calidad | Búsqueda en `HashMap` |
|---|---|
| Buena distribución | O(1) |
| Muchas colisiones | O(log n) desde Java 8 (árbol) |
| Constante | O(log n), y con mucho más costo constante |

Detalle de cómo lo usa el `HashMap` en [[11 - Internals de Colecciones]].

## 🧪 Tests

```java
@Test
void equals_cumple_el_contrato() {
    var a = new Cuenta("ACC-001", PEN);
    var b = new Cuenta("ACC-001", PEN);
    var c = new Cuenta("ACC-001", PEN);

    assertEquals(a, a);                                  // reflexiva
    assertEquals(a.equals(b), b.equals(a));              // simétrica
    assertTrue(a.equals(b) && b.equals(c) && a.equals(c)); // transitiva
    assertNotEquals(null, a);                            // no nula
}

@Test
void objetos_iguales_tienen_el_mismo_hashCode() {
    assertEquals(new Cuenta("ACC-001", PEN).hashCode(),
                 new Cuenta("ACC-001", PEN).hashCode());
}

@Test
void funciona_como_clave_de_un_mapa() {
    var mapa = new HashMap<Cuenta, String>();
    mapa.put(new Cuenta("ACC-001", PEN), "valor");
    assertEquals("valor", mapa.get(new Cuenta("ACC-001", PEN)));
}

@Test
void el_objeto_es_realmente_inmutable() {
    var lista = new ArrayList<>(List.of(feriado));
    var periodo = new Periodo(inicio, lista);
    lista.add(otroFeriado);
    assertEquals(1, periodo.feriados().size());          // la copia defensiva funcionó
    assertThrows(UnsupportedOperationException.class,
                 () -> periodo.feriados().add(otroFeriado));
}
```

```xml
<!-- EqualsVerifier: verifica el contrato completo automáticamente -->
<dependency>
    <groupId>nl.jqno.equalsverifier</groupId>
    <artifactId>equalsverifier</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@Test
void contrato_verificado() {
    EqualsVerifier.forClass(Cuenta.class)
        .withNonnullFields("numero", "moneda")
        .verify();      // comprueba simetría, transitividad, hashCode, mutabilidad...
}
```

> [!tip] `EqualsVerifier` es una línea que reemplaza diez tests
> Verifica todas las propiedades del contrato, incluidas las que casi nadie prueba (herencia, campos mutables, `hashCode` inconsistente). Vale la pena en cualquier clase que se use como clave.

## ⚠️ Errores Comunes

> [!warning] Los que producen bugs silenciosos
> - **Redefinir `equals` sin `hashCode`** (o al revés).
> - **Campos distintos** en `equals` y en `hashCode`.
> - **Campos mutables** en `hashCode`: el objeto se pierde en el `Set`.
> - **Extender una clase concreta** agregando estado: simetría rota.
> - **`getClass()`** con proxies de Hibernate.
> - **Id generado** en `equals` de entidades JPA.
> - **`record` con `BigDecimal`** sin normalizar la escala.
> - **`record` con arrays**: compara referencias.
> - **Sin copia defensiva** en un objeto que se declara inmutable.
> - **Devolver la colección interna** sin envolverla.

## 📌 Notas Rápidas
- La regla más simple para no equivocarse: **usá `record` cuando puedas**. El contrato viene resuelto y no se puede desincronizar.
- El caso de `BigDecimal` en un `record` es la trampa que más se ve en código financiero, y explica por qué `Dinero` normaliza la escala en el constructor compacto.
- La imposibilidad de extender una clase concreta preservando el contrato es una buena respuesta de entrevista: muestra que conocés el límite del modelo, no solo la receta.
- La inmutabilidad conecta con [[09 - Modelo de Memoria de Java]]: los campos `final` dan publicación segura, y eso elimina una familia entera de bugs de concurrencia.

## 🔗 Relaciones
- **Módulo**: [[Java Avanzado]]
- **Anterior**: [[09 - Modelo de Memoria de Java]]
- **Siguiente**: [[11 - Internals de Colecciones]]
- **Se conecta con**: [[09 - Records y Sealed Classes]], [[01 - Dinero en Software]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

