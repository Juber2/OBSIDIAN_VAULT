---
tags:
  - Java
  - Fundamentos
Creado: 2026-09-10
Relacionado:
  - Java Fundamentos
  - 05 - POO - Clases y Objetos
Palabras clave:
  - static, final, constante, bloque estatico, effectively final, inmutable, singleton
---

## 🧠 Concepto

Dos palabras clave que se aprenden de memoria y se entienden tarde:

| Palabra | Responde | En una línea |
|---|---|---|
| **`static`** | ¿de quién es? | pertenece a la **clase**, no a cada objeto |
| **`final`** | ¿puede cambiar? | **no se puede reasignar** |

Son independientes: podés tener `static` sin `final`, `final` sin `static`, las dos juntas o ninguna.

## 📌 `static` — pertenece a la clase

```java
public class Cuenta {
    private static int totalCuentas = 0;      // UNA sola para toda la clase
    private String numero;                     // UNA por cada objeto

    public Cuenta(String numero) {
        this.numero = numero;
        totalCuentas++;                        // se comparte entre todas las instancias
    }

    public static int getTotalCuentas() {      // se llama sin crear objeto
        return totalCuentas;
    }
}
```

```java
new Cuenta("ACC-001");
new Cuenta("ACC-002");

Cuenta.getTotalCuentas();      // 2 — se llama sobre la CLASE
```

```
Memoria:
  Cuenta (clase)  →  totalCuentas = 2        ← una sola copia
  objeto #1       →  numero = "ACC-001"
  objeto #2       →  numero = "ACC-002"
```

### Métodos estáticos

```java
public class Calculadora {
    public static int sumar(int a, int b) { return a + b; }
}

Calculadora.sumar(2, 3);       // sin instanciar nada
```

> [!danger] Un método `static` no puede tocar estado de instancia
> ```java
> public class Cuenta {
>     private String numero;
>     public static void imprimir() {
>         System.out.println(numero);    // ❌ no compila: ¿el numero de CUÁL objeto?
>     }
> }
> ```
> El método pertenece a la clase y puede llamarse sin que exista ningún objeto. No hay un `this` al que referirse.

### Bloques estáticos

```java
public class Configuracion {
    private static final Map<String, String> DEFECTOS;

    static {                                   // se ejecuta UNA vez, al cargar la clase
        DEFECTOS = new HashMap<>();
        DEFECTOS.put("timeout", "5000");
        DEFECTOS.put("reintentos", "3");
    }
}
```

Se ejecuta cuando la JVM carga la clase, antes de cualquier uso. Sirve para inicializaciones que no caben en una sola expresión.

> [!warning] Una excepción en un bloque estático es fatal
> Si falla, la JVM lanza `ExceptionInInitializerError` y **la clase queda inutilizable** para siempre en esa ejecución. Todo intento posterior de usarla da `NoClassDefFoundError`, que es un error confuso y difícil de rastrear hasta su causa real.

### `import static`

```java
import static java.lang.Math.max;
import static org.junit.jupiter.api.Assertions.*;

int mayor = max(a, b);         // en vez de Math.max(a, b)
assertEquals(esperado, real);  // en vez de Assertions.assertEquals(...)
```

Se usa mucho en tests. Fuera de ahí, con moderación: leer `max(a, b)` sin saber de dónde sale confunde.

## 🔒 `final` — no se puede reasignar

Aplica a tres cosas distintas:

```java
// 1 · Variable: no se puede reasignar
final int MAX = 100;
MAX = 200;                     // ❌ no compila

// 2 · Método: no se puede sobrescribir
public class Base {
    public final void validar() { ... }
}
public class Hija extends Base {
    public void validar() { ... }    // ❌ no compila
}

// 3 · Clase: no se puede heredar
public final class Dinero { ... }
public class MiDinero extends Dinero { }   // ❌ no compila
```

### La confusión central

> [!danger] `final` no significa inmutable
> ```java
> final List<String> lista = new ArrayList<>();
> lista.add("hola");             // ✅ PERMITIDO: la lista cambia
> lista = new ArrayList<>();     // ❌ prohibido: reasignar la referencia
> ```
> `final` congela **la referencia**, no el objeto al que apunta. Es de las confusiones más frecuentes.

```java
// Para que sea realmente inmutable
final List<String> lista = List.copyOf(origen);
lista.add("x");                // 💥 UnsupportedOperationException
```

Ver [[10 - equals hashCode e Inmutabilidad]] para inmutabilidad de verdad.

### `static final` = constante

```java
public class Limites {
    public static final BigDecimal LIMITE_DIARIO = new BigDecimal("15000.00");
    public static final int MAX_INTENTOS = 5;
    private static final Duration TIMEOUT = Duration.ofSeconds(3);
}
```

Convención: **MAYÚSCULAS con guiones bajos**. Es la señal visual de que es una constante.

> [!warning] Una constante que apunta a algo mutable no es constante
> ```java
> public static final List<String> ROLES = new ArrayList<>(List.of("ADMIN"));
> // Cualquiera puede hacer Limites.ROLES.add("HACKER") desde otra clase
>
> public static final List<String> ROLES = List.of("ADMIN", "OPERADOR");   // ✅
> ```

## 🪄 *Effectively final*

Desde Java 8, una variable local que **nunca se reasigna** se considera `final` aunque no lleve la palabra.

```java
String prefijo = "Cuenta: ";           // no se reasigna → effectively final

lista.forEach(c -> System.out.println(prefijo + c));   // ✅ la lambda puede usarla
```

```java
String prefijo = "Cuenta: ";
prefijo = "Cta: ";                     // ← se reasigna

lista.forEach(c -> System.out.println(prefijo + c));
// ❌ no compila: "local variables referenced from a lambda must be final or effectively final"
```

> [!tip] Por qué existe esta regla
> La lambda puede ejecutarse después, o en otro hilo. Java **copia el valor** de la variable en vez de compartir la referencia a la variable. Si la variable pudiera cambiar después, la lambda vería un valor viejo y sería imposible razonar sobre el resultado.
> La solución cuando necesitás acumular no es reasignar, sino usar un objeto mutable (`AtomicInteger`) o un `reduce`.

## ⚠️ El peligro: estado mutable estático

```java
// ❌ El error más caro de esta nota
@Service
public class ReporteService {
    private static List<Fila> acumulador = new ArrayList<>();

    public void agregar(Fila f) { acumulador.add(f); }
}
```

Tres problemas a la vez:

1. **Compartido entre todos los requests** — dos usuarios pisan los datos del otro.
2. **No es seguro para hilos** — `ArrayList` bajo concurrencia se corrompe.
3. **Nunca se libera** — la lista crece hasta agotar la memoria.

> [!danger] Una variable estática vive mientras viva la clase
> Y la clase vive mientras viva el classloader, que en una aplicación es toda la ejecución. Cualquier objeto referenciado desde un campo `static` **nunca se recolecta**. Es la causa clásica de fugas de memoria — ver [[03 - Memory Leaks]].

```java
// ✅ El estado vive en el método
public Reporte generar(List<Fila> filas) {
    List<Fila> acumulador = new ArrayList<>();
    ...
    return new Reporte(acumulador);
}
```

| Uso de `static` | ¿Está bien? |
|---|---|
| Constantes (`static final` inmutable) | ✅ |
| Métodos de utilidad sin estado | ✅ |
| Factory methods (`Dinero.de(...)`) | ✅ |
| Contadores y cachés mutables | ❌ casi siempre |
| Estado compartido entre requests | ❌ nunca |

## 🎯 Cuándo usar cada uno

```java
// static: pertenece al concepto, no a un objeto concreto
public static Dinero de(String monto, Moneda moneda) { ... }    // factory
public static boolean esValido(String iban) { ... }              // utilidad sin estado

// final en parámetros y variables locales: comunica intención
public void procesar(final ComandoTransferir cmd) {
    final Instant inicio = Instant.now();
    ...
}
```

> [!tip] `final` en campos es lo que más rinde
> Un campo `final` obliga a inicializarlo en el constructor y garantiza que nadie lo cambie después. Eso habilita la publicación segura entre hilos (ver [[09 - Modelo de Memoria de Java]]) y elimina toda una familia de bugs de estado inesperado.
> En parámetros y locales es más discutible: agrega ruido visual a cambio de poca cosa. Muchos equipos lo omiten ahí.

## 🧪 Para probar

```java
@Test
void el_contador_estatico_se_comparte_entre_instancias() {
    int antes = Cuenta.getTotalCuentas();
    new Cuenta("A");
    new Cuenta("B");
    assertEquals(antes + 2, Cuenta.getTotalCuentas());
}

@Test
void final_no_impide_modificar_el_objeto() {
    final List<String> lista = new ArrayList<>();
    lista.add("permitido");
    assertEquals(1, lista.size());
}

@Test
void una_lista_inmutable_si_lo_impide() {
    final List<String> lista = List.of("a");
    assertThrows(UnsupportedOperationException.class, () -> lista.add("b"));
}
```

## ⚠️ Errores Comunes

> [!warning] Los que aparecen en producción
> - **Estado mutable `static` en un bean de Spring**: compartido entre todos los requests.
> - **Creer que `final` hace inmutable el objeto.**
> - **`static final` sobre una colección mutable**: cualquiera la modifica.
> - **Bloque estático que puede lanzar excepción**: deja la clase inutilizable.
> - **Reasignar una variable que usa una lambda**: no compila y confunde.
> - **Abusar de `import static`**: no se sabe de dónde sale cada método.
> - **Métodos `static` para todo**: no se pueden sobrescribir ni sustituir en tests.

> [!warning] Los métodos `static` son difíciles de testear
> No se pueden sobrescribir ni reemplazar con un doble de prueba sin herramientas especiales. Si un servicio llama a `Utilidades.calcular(...)`, no hay forma limpia de aislarlo.
> Por eso las dependencias van por constructor y no por llamadas estáticas — ver [[02 - DI en Spring]].

## 📌 Notas Rápidas
- `static` responde **de quién es**; `final` responde **si puede cambiar**. Separar las dos preguntas evita casi toda la confusión.
- La combinación que casi siempre está bien es `private static final` sobre algo inmutable. La que casi nunca está bien es `static` sobre algo mutable.
- *Effectively final* es la razón por la que una lambda no te deja usar una variable que reasignás. El mensaje del compilador es explícito, pero el motivo (la lambda copia el valor) no.
- Un campo `static` mantiene vivo lo que referencia durante toda la ejecución: es la fuente más común de fugas de memoria en aplicaciones Java.

## 🔗 Relaciones
- **Módulo**: [[Java Fundamentos]]
- **Base**: [[05 - POO - Clases y Objetos]]
- **Se conecta con**: [[10 - equals hashCode e Inmutabilidad]], [[09 - Modelo de Memoria de Java]], [[03 - Memory Leaks]], [[01 - Singleton]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
