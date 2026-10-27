---
tags:
  - Java
  - Banca
  - Dominio
Creado: 2026-09-08
Relacionado:
  - Banca y Fintech
  - Java Fundamentos
Palabras clave:
  - BigDecimal, escala, redondeo, RoundingMode, minor units, value object, dinero
---

## 🧠 Concepto

El dinero parece un número y no lo es. Un importe es la combinación de **cantidad + moneda + escala + política de redondeo**, y las cuatro cosas importan. Tratarlo como un `double` es el error más caro y más común del software financiero.

## ❌ Por qué `double` y `float` están prohibidos

`double` es un binario de coma flotante IEEE 754. Puede representar exactamente `0.5` o `0.25` (potencias de 2), pero **no** `0.1` ni `0.20`, igual que en decimal no podés escribir 1/3 con dígitos finitos.

```java
System.out.println(0.1 + 0.2);          // 0.30000000000000004
System.out.println(0.1 + 0.2 == 0.3);   // false
System.out.println(1.03 - 0.42);        // 0.6100000000000001

double total = 0;
for (int i = 0; i < 10; i++) total += 0.10;
System.out.println(total);              // 0.9999999999999999
```

El error individual es minúsculo. El problema es que **se acumula y no se cancela**: en un cierre de día con millones de operaciones, esos residuos producen un descuadre que nadie puede explicar ni corregir.

> [!danger] El costo real
> Un descuadre de céntimos detiene un cierre contable. Alguien tiene que investigar de dónde salió, y como el error está repartido en millones de filas, no hay forma de rastrearlo. Se termina "ajustando" con un asiento manual — que es exactamente lo que un auditor no quiere ver.

## ✅ `BigDecimal`, con tres reglas

### Regla 1 — Construir siempre desde `String`

```java
new BigDecimal("0.1")   // exactamente 0.1  ✅
new BigDecimal(0.1)     // 0.1000000000000000055511151231257827021181583404541015625  ❌
```

El constructor `double` recibe un valor **ya corrompido**: el error se produjo antes de llegar a `BigDecimal`.

```java
BigDecimal.valueOf(0.1)   // usa Double.toString: da "0.1", pero sigue partiendo de un double
```

Si el dato viene de JSON o de la base como `BigDecimal`, perfecto. Si en algún punto pasó por `double`, ya se perdió precisión y no se recupera.

### Regla 2 — Comparar con `compareTo`, nunca con `equals`

```java
new BigDecimal("1.0").equals(new BigDecimal("1.00"));      // false ← compara ESCALA
new BigDecimal("1.0").compareTo(new BigDecimal("1.00"));   // 0     ← compara VALOR
```

`equals` considera el número de decimales parte de la identidad. Para dinero eso casi nunca es lo que querés.

```java
// Atajos legibles
if (monto.compareTo(BigDecimal.ZERO) > 0)   // ...o mejor:
if (monto.signum() > 0)                     // ✅ más claro y más rápido
```

> [!warning] Consecuencia en colecciones
> Como `HashSet` y `HashMap` usan `equals`/`hashCode`, un `Set<BigDecimal>` puede contener `1.0` y `1.00` como elementos distintos. Por eso el value object `Dinero` normaliza la escala en el constructor.

### Regla 3 — Dividir siempre con escala y modo de redondeo

```java
a.divide(b);                                // 💥 ArithmeticException si el resultado es periódico
a.divide(b, 2, RoundingMode.HALF_UP);       // ✅
```

`BigDecimal` se niega a adivinar: si no le decís cuántos decimales y cómo redondear, lanza excepción. Es deliberado.

| Modo | Comportamiento | Cuándo |
|---|---|---|
| `HALF_UP` | 2.5 → 3, -2.5 → -3 | el más común en importes comerciales |
| `HALF_EVEN` | 2.5 → 2, 3.5 → 4 | *banker's rounding*: no sesga en agregados grandes |
| `DOWN` | trunca hacia cero | intereses a favor de la entidad |
| `UP` | aleja de cero | comisiones |
| `FLOOR` / `CEILING` | hacia -∞ / +∞ | casos contables específicos |

> [!tip] `HALF_EVEN` para totales, `HALF_UP` para importes individuales
> `HALF_UP` siempre sube en el empate, así que en millones de operaciones el sesgo se acumula a favor de un lado. `HALF_EVEN` reparte los empates y el sesgo tiende a cero. Por eso es el default de IEEE y el que usa la contabilidad para consolidados.

## 💰 El value object

Un `BigDecimal` suelto no dice de qué moneda es, ni impide sumar soles con dólares. Encapsularlo resuelve las dos cosas:

```java
public record Dinero(BigDecimal monto, Moneda moneda) implements Comparable<Dinero> {

    public Dinero {
        Objects.requireNonNull(monto);
        Objects.requireNonNull(moneda);
        monto = monto.setScale(moneda.decimales(), RoundingMode.HALF_UP);  // normaliza
    }

    public static Dinero de(String monto, Moneda moneda) {
        return new Dinero(new BigDecimal(monto), moneda);
    }

    public Dinero mas(Dinero otro) {
        exigirMismaMoneda(otro);
        return new Dinero(monto.add(otro.monto), moneda);
    }

    public Dinero porcentaje(BigDecimal tasa) {
        return new Dinero(monto.multiply(tasa)
                               .setScale(moneda.decimales(), RoundingMode.HALF_UP), moneda);
    }

    private void exigirMismaMoneda(Dinero otro) {
        if (moneda != otro.moneda) throw new MonedaIncompatibleException(moneda, otro.moneda);
    }
}
```

Con esto, `sumar soles con dólares` deja de ser un bug posible: **no compila mentalmente y falla en tiempo de ejecución con un mensaje claro**.

## 🔢 La alternativa: enteros en unidades menores

Guardar céntimos en un `long` en lugar de decimales:

```java
long saldoEnCentimos = 150_050L;   // S/ 1500.50
```

| | `BigDecimal` | `long` en unidades menores |
|---|---|---|
| Exactitud | ✅ | ✅ |
| Velocidad | más lento | muy rápido |
| Legibilidad | natural | hay que dividir para mostrar |
| División y porcentajes | directa | requiere cuidado extra |
| Monedas de 0 o 3 decimales | soportado por la escala | hay que conocer el exponente |
| Riesgo de desbordamiento | ninguno | `long` se desborda cerca de 9,2×10¹⁸ |

Lo usan sistemas de altísimo volumen (Stripe expone sus importes así en la API). Para una aplicación de negocio, **`BigDecimal` es la elección por defecto**: la diferencia de rendimiento es irrelevante frente al costo de un error.

## 🗄️ En la base de datos

```sql
monto NUMERIC(19,4) NOT NULL   -- ✅ exacto
monto DOUBLE PRECISION         -- ❌ mismo problema que en Java
monto FLOAT                    -- ❌
monto MONEY                    -- ⚠️ tipo propio de Postgres: escala fija, mal con multi-moneda
```

**Por qué escala 4 y no 2:** los importes finales tienen 2 decimales, pero los cálculos intermedios (intereses diarios, prorrateos, conversiones) necesitan más precisión antes de redondear. Con escala 2 en la base, cada paso intermedio redondea y el error se acumula.

| Tipo SQL | Tipo Java (JDBC) |
|---|---|
| `NUMERIC` / `DECIMAL` | `BigDecimal` ✅ |
| `DOUBLE PRECISION` | `Double` ❌ |
| `BIGINT` (unidades menores) | `long` ✅ |

## 🌐 En la API

```java
public record MovimientoResponse(BigDecimal monto, String moneda) {}
```

```json
{ "monto": 1500.50, "moneda": "PEN" }
```

> [!warning] Cuidado con el cliente JavaScript
> JavaScript solo tiene `number`, que es un `double` IEEE 754. Un importe grande enviado como número JSON puede perder precisión en el navegador. Si el cliente es JS y los montos son grandes, enviá el importe como **string**:
> ```json
> { "monto": "1500.50", "moneda": "PEN" }
> ```
> Es lo que hacen varias APIs financieras justamente por esto.

Configuración de Jackson para no arruinar la precisión:

```java
@Bean
public Jackson2ObjectMapperBuilderCustomizer decimalesExactos() {
    return builder -> builder
        .featuresToEnable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
        .featuresToDisable(JsonGenerator.Feature.WRITE_BIGDECIMAL_AS_PLAIN);
}
```

## 🧪 Tests que valen la pena

```java
@Test
void sumar_centimos_no_pierde_precision() {
    Dinero total = IntStream.range(0, 100)
        .mapToObj(i -> Dinero.de("0.01", PEN))
        .reduce(Dinero.cero(PEN), Dinero::mas);

    assertEquals(Dinero.de("1.00", PEN), total);
}

@Test
void el_mismo_valor_con_distinta_escala_es_igual() {
    assertEquals(Dinero.de("1.0", PEN), Dinero.de("1.00", PEN));
}

@Test
void no_se_pueden_mezclar_monedas() {
    assertThrows(MonedaIncompatibleException.class,
        () -> Dinero.de("10", PEN).mas(Dinero.de("10", USD)));
}
```

Y la regla congelada en un test de arquitectura, para que no dependa de que alguien se acuerde:

```java
@ArchTest
static final ArchRule ningun_importe_usa_double =
    noFields().should().haveRawType(double.class).orShould().haveRawType(Double.class);
```

## ⚠️ Errores Comunes

> [!warning] Los seis clásicos
> - `new BigDecimal(0.1)` en lugar de `new BigDecimal("0.1")`.
> - Comparar importes con `equals` en vez de `compareTo`.
> - `divide` sin escala ni `RoundingMode` → `ArithmeticException` en producción.
> - Redondear en cada paso intermedio en vez de solo al final.
> - Guardar como `FLOAT` en la base aunque en Java sea `BigDecimal`.
> - Un `BigDecimal` sin moneda asociada: nada impide sumar PEN con USD.

## 📌 Notas Rápidas
- La escala de una moneda no siempre es 2: el yen japonés (JPY) usa 0 decimales y el dinar kuwaití (KWD) usa 3. Un sistema que asume 2 fijo se rompe al internacionalizarse — por eso la escala vive en el enum `Moneda`.
- `BigDecimal` es inmutable: `monto.add(otro)` **devuelve** un nuevo valor, no modifica el original. Olvidar asignar el resultado es un error silencioso frecuente.
- Si en una entrevista te preguntan por qué no se usa `double` para dinero, la respuesta completa incluye *por qué*: representación binaria de fracciones decimales, y acumulación del error en agregados.
- Se aplica en [[Fase 0 - Ledger en Java Puro]] y se verifica en el cierre de [[Fase 8 - Batch y Cierre de Dia]].

## 🔗 Relaciones
- **Módulo**: [[Banca y Fintech]]
- **Siguiente**: [[02 - Multimoneda y Tipos de Cambio]]
- **Se conecta con**: [[03 - Ledger y Partida Doble]], [[Java-errores-comunes]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

