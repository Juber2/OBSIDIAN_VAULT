---
tags:
  - Java
  - Testing
Creado: 2026-05-21
Relacionado:
  - Testing en Java
  - JUnit 5
Palabras clave:
  - TDD, test driven development, red-green-refactor, ciclo TDD
---

## 🧠 Concepto

**TDD (Test Driven Development)** es una metodología donde escribes el test **antes** que el código de producción.

## Ciclo Red → Green → Refactor

```
1. RED    — Escribir un test que falla (la funcionalidad no existe aún)
2. GREEN  — Escribir el mínimo código para que el test pase
3. REFACTOR — Mejorar el código sin romper los tests
```

## Ejemplo paso a paso

### Paso 1 — RED: escribir el test

```java
// Quiero una clase que calcule el precio con IVA
class CalculadoraPrecioTest {

    @Test
    void calcularPrecioConIVA_precio100_debe115() {
        CalculadoraPrecio calc = new CalculadoraPrecio();
        double resultado = calc.conIVA(100.0);
        assertEquals(115.0, resultado, 0.01);
    }
}
// ❌ No compila — CalculadoraPrecio no existe
```

### Paso 2 — GREEN: mínimo código que hace pasar el test

```java
public class CalculadoraPrecio {
    public double conIVA(double precio) {
        return precio * 1.15; // 15% IVA
    }
}
// ✅ Test pasa
```

### Paso 3 — REFACTOR: mejorar

```java
public class CalculadoraPrecio {
    private static final double IVA = 0.15;

    public double conIVA(double precio) {
        validar(precio);
        return precio * (1 + IVA);
    }

    private void validar(double precio) {
        if (precio < 0) throw new IllegalArgumentException("Precio no puede ser negativo");
    }
}

// Agregar test para la validación
@Test
void conIVA_precioNegativo_debeLanzarExcepcion() {
    assertThrows(IllegalArgumentException.class, () -> calc.conIVA(-10));
}
```

## Principios del TDD

```
Regla 1: No escribir código de producción sin un test fallando que lo justifique
Regla 2: No escribir más test del necesario para que falle
Regla 3: No escribir más código del necesario para hacer pasar el test
```

## FIRST — propiedades de buenos tests

| Letra | Propiedad | Descripción |
|-------|-----------|-------------|
| **F** | Fast | Rápidos — milliseconds |
| **I** | Isolated | Aislados — no dependen entre sí |
| **R** | Repeatable | Repetibles — mismo resultado siempre |
| **S** | Self-validating | Auto-validantes — pass o fail, sin interpretación |
| **T** | Timely | A tiempo — antes o con el código |

## Estructura AAA (Arrange, Act, Assert)

```java
@Test
void registrar_usuarioNuevo_debeRetornarUsuarioGuardado() {
    // Arrange — preparar datos y mocks
    RegistroRequest request = new RegistroRequest("Juber", "juber@email.com");
    when(repository.save(any())).thenReturn(usuarioGuardado);

    // Act — ejecutar lo que se está testeando
    UsuarioResponse resultado = servicio.registrar(request);

    // Assert — verificar el resultado
    assertNotNull(resultado);
    assertEquals("Juber", resultado.getNombre());
    verify(repository).save(any());
}
```

## Cuándo TDD tiene más valor

- Lógica de negocio compleja
- Algoritmos con múltiples casos borde
- Reglas de validación
- Cálculos financieros o matemáticos

## ⚠️ TDD no es para todo

No es obligatorio hacer TDD en:
- Prototipos exploratorios
- Código de infraestructura/configuración
- Tests de integración (son posteriores)

## 📌 Notas Rápidas
- El valor real de TDD no es "tener tests" — es que el diseño del código termina siendo más desacoplado, porque escribir el test primero obliga a pensar en cómo se va a usar la clase antes de implementarla.
- "Mínimo código para pasar el test" a veces significa código deliberadamente tonto (`return 115.0;` hardcodeado) — se generaliza recién cuando un segundo test lo obliga. Es contraintuitivo pero es la disciplina correcta.
- TDD estricto es más común en katas y lógica de negocio pura; en la práctica de equipo, muchos hacen "test-after" (escriben el código y el test casi en simultáneo) sin perder la disciplina de cobertura.
- No confundir TDD (metodología de cómo escribís código) con Testing (la práctica general de tener tests) — se puede tener buena cobertura sin haber hecho TDD estricto.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

