---
tags:
  - Java
  - Banca
  - Dominio
Creado: 2026-09-08
Relacionado:
  - Banca y Fintech
  - 01 - Dinero en Software
Palabras clave:
  - multimoneda, tipo de cambio, spread, ISO 4217, conversion, exposicion cambiaria
---

## 🧠 Concepto

Un sistema multi-moneda no es "el mismo sistema con un campo `moneda`". Cambia el modelo: los saldos dejan de ser sumables entre sí, las conversiones necesitan una tasa con fecha, y el redondeo de la conversión tiene que ir a algún lado contable.

## 🔤 ISO 4217

El estándar que define los códigos de moneda. Tres piezas:

| Elemento | Ejemplo | Uso |
|---|---|---|
| Código alfabético | `PEN`, `USD`, `EUR`, `JPY` | el que se usa en APIs y bases |
| Código numérico | `604`, `840`, `978`, `392` | ISO 8583 y sistemas de tarjetas |
| Exponente (decimales) | 2, 2, 2, **0** | cuántos decimales tiene la moneda |

```java
public enum Moneda {
    PEN("604", "S/",  2),
    USD("840", "$",   2),
    EUR("978", "€",   2),
    JPY("392", "¥",   0),    // sin decimales
    KWD("414", "KD",  3);    // tres decimales

    private final String codigoNumerico;
    private final String simbolo;
    private final int decimales;
    // ...
}
```

> [!warning] Nunca asumas 2 decimales
> El yen no tiene decimales: `¥ 1000.00` es incorrecto, son `¥ 1000`. El dinar kuwaití tiene tres. Un sistema que hardcodea 2 se rompe al internacionalizarse — y peor, redondea mal sin avisar.

Java ya trae una implementación de ISO 4217 si no querés mantener el enum:

```java
Currency pen = Currency.getInstance("PEN");
pen.getDefaultFractionDigits();   // 2
pen.getNumericCode();             // 604
```

## 🚫 La regla que ordena todo

**Los importes de monedas distintas no se suman.** Nunca, bajo ninguna circunstancia, sin una conversión explícita con una tasa identificable.

```java
Dinero soles   = Dinero.de("100", PEN);
Dinero dolares = Dinero.de("100", USD);

soles.mas(dolares);   // 💥 MonedaIncompatibleException — y está bien
```

Si el value object permite la suma silenciosa, tarde o temprano alguien va a sumar el saldo de todas las cuentas de un cliente y obtener un número sin significado.

### El saldo de un cliente es un mapa, no un número

```java
public record SaldoConsolidado(Map<Moneda, Dinero> porMoneda) {

    /** La conversión a una moneda de referencia es una VISTA, no el saldo real */
    public Dinero equivalenteEn(Moneda destino, TipoDeCambio tc) {
        return porMoneda.values().stream()
            .map(d -> tc.convertir(d, destino))
            .reduce(Dinero.cero(destino), Dinero::mas);
    }
}
```

> [!tip] La conversión para mostrar nunca se persiste
> "Tu patrimonio total: S/ 12.450" es una vista calculada al momento con la tasa del momento. Si la guardás, mañana estará mal. La verdad son los saldos por moneda.

## 💱 El tipo de cambio

### Compra, venta y spread

Una casa de cambio no tiene "un" tipo de cambio: tiene dos.

```
USD/PEN   compra 3.720   ← te compra dólares a este precio
          venta  3.745   ← te vende dólares a este precio
          spread 0.025   ← el margen de la entidad
```

**Cuál se aplica depende de la dirección de la operación desde la perspectiva de la entidad**, no del cliente:

| El cliente… | La entidad… | Tasa que aplica |
|---|---|---|
| entrega USD y recibe PEN | compra USD | **compra** (3.720) |
| entrega PEN y recibe USD | vende USD | **venta** (3.745) |

```java
public record TipoDeCambio(
        Moneda base,          // USD
        Moneda cotizada,      // PEN
        BigDecimal compra,
        BigDecimal venta,
        Instant vigenteDesde,
        String fuente         // trazabilidad: quién la publicó
) {
    public BigDecimal tasaPara(Moneda origen, Moneda destino) {
        if (origen == base && destino == cotizada) return compra;   // el cliente entrega base
        if (origen == cotizada && destino == base) return venta;
        throw new ParDeMonedasNoSoportadoException(origen, destino);
    }
}
```

> [!danger] Guardá siempre la tasa aplicada en la operación
> No basta con guardar los dos importes. Meses después, un cliente reclama y hay que poder demostrar **qué tasa se usó, de qué fuente y con qué vigencia**. Si solo tenés "entraron 100 USD, salieron 372 PEN", no podés justificar nada.

### La conversión

```java
public Dinero convertir(Dinero origen, Moneda destino, TipoDeCambio tc) {
    if (origen.moneda() == destino) return origen;

    BigDecimal tasa = tc.tasaPara(origen.moneda(), destino);

    BigDecimal convertido = origen.monto()
        .multiply(tasa)                                          // sin redondear todavía
        .setScale(destino.decimales(), RoundingMode.HALF_UP);     // redondeo AL FINAL

    return new Dinero(convertido, destino);
}
```

**El redondeo va solo al final.** Si redondeás la tasa, después el producto y después el total, acumulás tres errores en vez de uno.

## ⚖️ El asiento de una conversión

Acá está el detalle que casi nadie modela bien: **una conversión no cuadra en una sola moneda**. Salen 100 USD y entran 372 PEN — no son la misma cantidad de nada.

La forma correcta es tratarla como **dos asientos contra una cuenta puente de posición de cambio**:

```
Asiento 1 (en USD)
  cuenta cliente USD        -100.00 USD
  posición de cambio USD    +100.00 USD    → cuadra en USD ✅

Asiento 2 (en PEN)
  posición de cambio PEN    -372.00 PEN
  cuenta cliente PEN        +372.00 PEN    → cuadra en PEN ✅
```

Cada asiento cuadra en su propia moneda. La cuenta de posición de cambio acumula la **exposición cambiaria** de la entidad: cuántos dólares tiene comprometidos y a qué precio.

> [!tip] Por eso el cuadre se verifica por moneda, no global
> La invariante correcta no es "la suma de todos los apuntes da cero", sino **"la suma de los apuntes de cada moneda da cero"**. Es exactamente lo que hace `Asiento.exigirQueCuadre()` en [[Fase 0 - Ledger en Java Puro]].

```sql
SELECT moneda, SUM(monto) AS suma
FROM apunte
GROUP BY moneda
HAVING SUM(monto) <> 0;     -- ninguna fila = todo cuadra
```

## 🕐 Vigencia de las tasas

Una tasa no es un valor: es un valor **con un período de validez**.

```sql
CREATE TABLE tipo_cambio (
    id              BIGSERIAL PRIMARY KEY,
    moneda_base     VARCHAR(3)    NOT NULL,
    moneda_cotizada VARCHAR(3)    NOT NULL,
    compra          NUMERIC(19,8) NOT NULL,   -- más decimales que un importe
    venta           NUMERIC(19,8) NOT NULL,
    vigente_desde   TIMESTAMPTZ   NOT NULL,
    vigente_hasta   TIMESTAMPTZ,              -- null = vigente
    fuente          VARCHAR(40)   NOT NULL,
    UNIQUE (moneda_base, moneda_cotizada, vigente_desde)
);

CREATE INDEX idx_tc_vigente ON tipo_cambio(moneda_base, moneda_cotizada, vigente_desde DESC);
```

> [!warning] `NUMERIC(19,8)` para tasas, no `(19,2)`
> Una tasa de cambio necesita más decimales que un importe. Redondearla a 2 introduce un error que se multiplica por cada importe convertido.

Y en la operación se guarda la tasa **efectivamente aplicada**, no una referencia a la tabla:

```sql
ALTER TABLE asiento
    ADD COLUMN tasa_aplicada   NUMERIC(19,8),
    ADD COLUMN tipo_cambio_id  BIGINT REFERENCES tipo_cambio(id);
```

Si mañana alguien corrige una tasa mal cargada, las operaciones ya hechas conservan la que realmente se usó.

## 🔄 Pares y conversión cruzada

No todas las combinaciones tienen cotización directa. `PEN/JPY` probablemente no exista, pero sí `USD/PEN` y `USD/JPY`.

```java
public Dinero convertirCruzado(Dinero origen, Moneda destino) {
    return buscarTasaDirecta(origen.moneda(), destino)
        .map(tc -> convertir(origen, destino, tc))
        .orElseGet(() -> {
            // vía moneda pivote (normalmente USD)
            Dinero enPivote = convertir(origen, PIVOTE, tasa(origen.moneda(), PIVOTE));
            return convertir(enPivote, destino, tasa(PIVOTE, destino));
        });
}
```

> [!danger] La conversión cruzada acumula spread dos veces
> El cliente paga el margen del primer par y el del segundo. Es correcto contablemente, pero hay que **mostrarlo**: si el cliente ve una tasa efectiva peor que la publicada sin explicación, es un reclamo asegurado.

## 🧪 Tests

```java
@Test
void la_conversion_genera_dos_asientos_que_cuadran_por_separado() {
    var resultado = servicio.convertir(cuentaUsd, cuentaPen, Dinero.de("100", USD));

    assertEquals(0, sumaDeApuntes(resultado.asientos(), USD).signum());
    assertEquals(0, sumaDeApuntes(resultado.asientos(), PEN).signum());
}

@Test
void comprar_y_vender_de_inmediato_deja_al_cliente_con_menos_por_el_spread() {
    // no es un bug: es el margen de la entidad, y debe ser exactamente el esperado
}

@Test
void una_moneda_sin_decimales_no_admite_fracciones() {
    assertEquals(Dinero.de("1000", JPY), Dinero.de("1000.4", JPY));   // se normaliza a 1000
}

@Test
void la_operacion_guarda_la_tasa_aplicada() {
    assertNotNull(asiento.tasaAplicada());
}
```

## ⚠️ Errores Comunes

> [!warning] Los que se ven en producción
> - **Sumar monedas distintas** porque el saldo se modeló como un `BigDecimal` suelto.
> - **Una sola tasa** en vez de compra/venta: la entidad pierde el spread o se lo cobra al revés.
> - **No guardar la tasa aplicada**, imposibilitando justificar la operación después.
> - **Redondear la tasa** antes de multiplicar.
> - **Asumir 2 decimales** para todas las monedas.
> - **Verificar el cuadre global** en vez de por moneda: da cero por casualidad o nunca da cero.
> - **Convertir para mostrar y persistir el resultado** como si fuera un saldo.

## 📌 Notas Rápidas
- La cuenta de posición de cambio no es un artificio contable: es donde la entidad ve su exposición real. Si compró muchos dólares y el tipo de cambio se mueve, ahí está la ganancia o la pérdida.
- El redondeo de una conversión genera céntimos que tienen que ir a alguna cuenta — típicamente una de "diferencias de redondeo". Si no la tenés, esos céntimos son tu descuadre en el cierre.
- En Perú, el tipo de cambio de referencia lo publica la SBS; en otros países es el banco central. Una entidad suele usar esa referencia más su propio spread.
- Se aplica en [[Fase 8 - Batch y Cierre de Dia]] al conciliar y en [[09 - Conciliacion y Cuadre]].

## 🔗 Relaciones
- **Módulo**: [[Banca y Fintech]]
- **Anterior**: [[01 - Dinero en Software]]
- **Siguiente**: [[03 - Ledger y Partida Doble]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

