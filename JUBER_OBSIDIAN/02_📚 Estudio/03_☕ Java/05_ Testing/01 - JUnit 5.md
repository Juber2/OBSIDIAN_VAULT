---
tags:
  - Java
  - Testing
Creado: 2026-05-21
Relacionado:
  - Testing en Java
Palabras clave:
  - JUnit 5, test, assertions, @Test, @BeforeEach, @ParameterizedTest, lifecycle
---

## 🧠 Concepto

**JUnit 5** es el framework estándar de testing en Java. Compuesto por tres módulos:
- **JUnit Platform** — motor de ejecución
- **JUnit Jupiter** — API de escritura de tests (lo que usas)
- **JUnit Vintage** — compatibilidad con JUnit 4

## Dependencia (Maven)

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
</dependency>
```

## Test básico

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class CalculadoraTest {

    private Calculadora calc;

    @BeforeEach
    void setUp() {
        calc = new Calculadora(); // se ejecuta antes de CADA test
    }

    @AfterEach
    void tearDown() {
        // limpieza después de cada test
    }

    @BeforeAll
    static void setUpClass() {
        // una vez antes de todos los tests
    }

    @AfterAll
    static void tearDownClass() {
        // una vez después de todos los tests
    }

    @Test
    void sumar_dosNumeros_debeRetornarSuma() {
        int resultado = calc.sumar(3, 5);
        assertEquals(8, resultado);
    }

    @Test
    void dividir_entrecero_debeLanzarExcepcion() {
        assertThrows(ArithmeticException.class, () -> calc.dividir(10, 0));
    }

    @Test
    @Disabled("Pendiente de implementar")
    void testPendiente() { }
}
```

## Assertions principales

```java
// Básicas
assertEquals(esperado, actual);
assertEquals(esperado, actual, "mensaje de fallo");
assertNotEquals(noEsperado, actual);
assertTrue(condicion);
assertFalse(condicion);
assertNull(objeto);
assertNotNull(objeto);

// Objetos
assertSame(obj1, obj2);    // misma referencia
assertNotSame(obj1, obj2);

// Arrays y colecciones
assertArrayEquals(new int[]{1,2,3}, resultado);
assertIterableEquals(List.of(1,2,3), listaResultado);

// Múltiples assertions (no se detiene en el primero que falla)
assertAll(
    () -> assertEquals("Juber", usuario.getNombre()),
    () -> assertEquals("juber@email.com", usuario.getEmail()),
    () -> assertTrue(usuario.isActivo())
);

// Excepciones
Exception ex = assertThrows(IllegalArgumentException.class, () -> {
    servicio.crear(null);
});
assertEquals("nombre no puede ser null", ex.getMessage());

// Timeout
assertTimeout(Duration.ofMillis(100), () -> {
    // debe completarse en menos de 100ms
    operacion.ejecutar();
});
```

## Tests parametrizados

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3, 4, 5})
void esMenorQue10(int numero) {
    assertTrue(numero < 10);
}

@ParameterizedTest
@CsvSource({
    "3, 5, 8",
    "0, 0, 0",
    "-1, 1, 0"
})
void sumar(int a, int b, int esperado) {
    assertEquals(esperado, calc.sumar(a, b));
}

@ParameterizedTest
@MethodSource("proveerDatos")
void testConMetodo(String input, int esperado) {
    assertEquals(esperado, servicio.procesar(input));
}

static Stream<Arguments> proveerDatos() {
    return Stream.of(
        Arguments.of("hola", 4),
        Arguments.of("mundo", 5)
    );
}

@ParameterizedTest
@EnumSource(value = TipoPago.class, names = {"EFECTIVO", "TARJETA"})
void procesarPago(TipoPago tipo) {
    assertTrue(servicio.aceptaTipo(tipo));
}
```

## Anotaciones de ciclo de vida y condiciones

```java
@Test
@DisplayName("Debe calcular el precio con descuento correctamente")
void testConNombreLegible() { }

@Test
@Tag("integration")
void testDeIntegracion() { }

@Test
@EnabledOnOs(OS.LINUX)
void soloEnLinux() { }

@Test
@EnabledIfSystemProperty(named = "env", matches = "ci")
void soloEnCI() { }

@Nested
class CuandoElUsuarioEsAdmin {
    @Test
    void puedeBorrar() { }
}
```

## ⚠️ Errores Comunes

> [!warning] No hacer `@BeforeAll` sin `static`
> `@BeforeAll` requiere que el método sea `static` a menos que el test esté anotado con `@TestInstance(Lifecycle.PER_CLASS)`.

> [!tip] Convención de nombres
> `metodo_escenario_resultadoEsperado` — hace los tests más legibles y autodescriptivos.

## 📌 Notas Rápidas
