---
tags:
  - Java
  - Fundamentos
  - POO
Creado: 2026-09-10
Relacionado:
  - Java Fundamentos
  - 12 - static y final
Palabras clave:
  - clase anidada, clase interna, anonima, local, lambda, builder, fuga de memoria
---

## 🧠 Concepto

Java permite declarar una clase **dentro de otra**. Hay cuatro variantes y la diferencia entre ellas —sobre todo entre las dos primeras— tiene consecuencias reales de memoria.

| Tipo | Declaración | ¿Guarda referencia al exterior? |
|---|---|---|
| **Anidada estática** | `static class X` dentro de otra | ❌ no |
| **Interna** | `class X` dentro de otra | ✅ **sí** |
| **Local** | dentro de un método | ✅ sí |
| **Anónima** | `new Interfaz() { ... }` | ✅ sí |

## 1️⃣ Anidada estática

Es una clase normal que vive dentro de otra solo por organización.

```java
public class Cuenta {

    private String numero;

    public static class Builder {          // ← static
        private String numero;

        public Builder numero(String n) { this.numero = n; return this; }
        public Cuenta build() { return new Cuenta(numero); }
    }
}
```

```java
Cuenta cuenta = new Cuenta.Builder()       // no hace falta una Cuenta previa
    .numero("ACC-001")
    .build();
```

> [!tip] Es la que deberías usar en el 90% de los casos
> No guarda referencia al objeto externo, así que no tiene el problema de memoria de las internas. Sirve para agrupar una clase auxiliar con la clase que la usa: builders, nodos de una estructura, DTOs internos.
> Es la que usa el patrón [[04 - Builder]].

## 2️⃣ Interna (no estática)

Sin `static`, la clase **necesita una instancia de la externa** para existir, y guarda una referencia a ella.

```java
public class Libro {

    private List<Asiento> asientos = new ArrayList<>();

    public class Iterador {                 // ← sin static
        private int indice = 0;

        public Asiento siguiente() {
            return asientos.get(indice++);  // accede al campo del EXTERNO directamente
        }
    }
}
```

```java
Libro libro = new Libro();
Libro.Iterador it = libro.new Iterador();   // sintaxis rara, y es la señal de alerta
```

```java
// Desde adentro, para referirse al objeto externo
public class Externa {
    private String nombre = "externa";

    public class Interna {
        private String nombre = "interna";

        public void imprimir() {
            System.out.println(nombre);              // "interna"
            System.out.println(this.nombre);         // "interna"
            System.out.println(Externa.this.nombre); // "externa"  ← sintaxis calificada
        }
    }
}
```

> [!danger] La referencia oculta es una fuente real de fugas de memoria
> Una instancia de `Iterador` **mantiene vivo** el `Libro` completo, aunque solo use un campo. Si guardás el iterador en una lista de larga vida —un caché, un registro de escuchas—, el `Libro` entero y todos sus asientos quedan sin poder recolectarse.
>
> ```java
> // ❌ El listener guarda una referencia implícita a toda la ventana
> public class Ventana {
>     private byte[] datosPesados = new byte[100_000_000];
>
>     public class Escucha implements EventListener { ... }
> }
> registroGlobal.add(ventana.new Escucha());   // los 100 MB nunca se liberan
> ```

**La regla práctica: si la clase anidada no necesita acceder al estado de la externa, hacela `static`.** Es una palabra que cambia el comportamiento de memoria.

## 3️⃣ Local

Declarada dentro de un método. Casi no se usa.

```java
public List<Resultado> procesar(List<Dato> datos) {

    class Acumulador {                      // solo existe en este método
        int total = 0;
        void sumar(int v) { total += v; }
    }

    var acc = new Acumulador();
    datos.forEach(d -> acc.sumar(d.valor()));
    return List.of(new Resultado(acc.total));
}
```

Se usa cuando necesitás una estructura auxiliar que no tiene sentido fuera del método. En la práctica, casi siempre conviene un `record` local (Java 16+) o directamente una clase aparte.

```java
public void procesar() {
    record Par(String clave, int valor) { }     // record local: mucho más limpio
    ...
}
```

## 4️⃣ Anónima

Implementa una interfaz o extiende una clase **en el mismo lugar donde la usás**, sin darle nombre.

```java
Comparator<Cuenta> porSaldo = new Comparator<Cuenta>() {
    @Override
    public int compare(Cuenta a, Cuenta b) {
        return a.saldo().compareTo(b.saldo());
    }
};
```

Con una interfaz funcional (un solo método abstracto), la lambda hace lo mismo:

```java
Comparator<Cuenta> porSaldo = (a, b) -> a.saldo().compareTo(b.saldo());
```

### Cuándo la anónima sigue siendo necesaria

```java
// 1 · La interfaz tiene MÁS de un método abstracto
var escucha = new MouseListener() {
    @Override public void mouseClicked(MouseEvent e) { ... }
    @Override public void mousePressed(MouseEvent e) { ... }
    // ... la lambda no puede: solo sirve para interfaces funcionales
};

// 2 · Necesitás estado propio
var contador = new Runnable() {
    private int veces = 0;                   // la lambda no puede tener campos
    @Override public void run() { veces++; }
};

// 3 · Extendés una CLASE, no una interfaz
var tarea = new TimerTask() {
    @Override public void run() { ... }
};
```

## ⚡ Lambda vs clase anónima

| | Clase anónima | Lambda |
|---|---|---|
| Genera un `.class` aparte | ✅ (`Externa$1.class`) | ❌ (`invokedynamic`) |
| Guarda referencia a la externa | ✅ **siempre** | solo si usa `this` o un campo |
| `this` dentro se refiere a… | la clase anónima | la clase **externa** |
| Puede tener campos y estado | ✅ | ❌ |
| Interfaces con varios métodos | ✅ | ❌ |

> [!warning] `this` significa cosas distintas en cada una
> ```java
> public class Servicio {
>     private String nombre = "servicio";
>
>     public void ejecutar() {
>         Runnable anonima = new Runnable() {
>             public void run() { System.out.println(this); }   // la clase ANÓNIMA
>         };
>         Runnable lambda = () -> System.out.println(this);     // el SERVICIO
>     }
> }
> ```
> Es una diferencia sutil que aparece al migrar código viejo de anónimas a lambdas.

> [!tip] La lambda no siempre captura la instancia externa
> Una lambda que no usa `this` ni ningún campo de la clase es **sin estado** y la JVM puede reutilizar una única instancia. Una clase anónima siempre crea un objeto nuevo que referencia al externo.
> En código de alto volumen esa diferencia importa.

## 🎯 Cuál usar

```
¿Necesito acceder al estado del objeto externo?
├── NO  → clase anidada ESTÁTICA   (o una clase aparte, si crece)
└── SÍ  → ¿es una implementación corta de una interfaz?
         ├── SÍ, interfaz funcional        → lambda
         ├── SÍ, varios métodos o estado   → clase anónima
         └── NO                             → clase interna (con cuidado)
```

| Caso | Elección |
|---|---|
| Builder | anidada estática |
| Nodo de una estructura de datos | anidada estática |
| DTO interno de un servicio | anidada estática o `record` |
| Comparador de una línea | lambda |
| Manejador de eventos con varios métodos | anónima |
| Iterador que recorre el estado del externo | interna |

## 🧪 Para probar

```java
@Test
void una_anidada_estatica_no_necesita_instancia_externa() {
    var builder = new Cuenta.Builder();      // sin crear una Cuenta antes
    assertNotNull(builder.numero("ACC-001").build());
}

@Test
void una_interna_si_la_necesita() {
    Libro libro = new Libro();
    Libro.Iterador it = libro.new Iterador();  // la sintaxis lo delata
    assertNotNull(it);
}

@Test
void this_en_una_lambda_es_la_clase_externa() {
    // documenta la diferencia con la clase anónima
}
```

## ⚠️ Errores Comunes

> [!warning] Los que importan
> - **Clase interna donde correspondía estática**: mantiene vivo el objeto externo.
> - **Guardar una instancia de clase interna** en una colección de larga vida: fuga de memoria.
> - **Clase anónima para una interfaz funcional**: la lambda es más corta y más liviana.
> - **Confundir `this`** al pasar de anónima a lambda.
> - **Clases anidadas gigantes**: si supera unas decenas de líneas, va en su propio archivo.
> - **Clase local** donde un `record` local sería más claro.

## 📌 Notas Rápidas
- La única palabra que separa una clase anidada segura de una fuga de memoria es **`static`**. Es el aprendizaje central de esta nota.
- La sintaxis `objeto.new Interna()` es fea a propósito: es la señal de que estás usando una clase interna y de que hay una referencia implícita en juego.
- En código moderno, las anónimas casi desaparecieron por las lambdas. Siguen apareciendo en código viejo, en `TimerTask` y en interfaces de varios métodos.
- Se aplica directamente en [[04 - Builder]], que usa una anidada estática, y se relaciona con [[03 - Memory Leaks]].

## 🔗 Relaciones
- **Módulo**: [[Java Fundamentos]]
- **Base**: [[05 - POO - Clases y Objetos]], [[07 - Interfaces y Abstracciones]]
- **Se conecta con**: [[12 - static y final]], [[02 - Lambdas y Functional Interfaces]], [[04 - Builder]], [[03 - Memory Leaks]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
