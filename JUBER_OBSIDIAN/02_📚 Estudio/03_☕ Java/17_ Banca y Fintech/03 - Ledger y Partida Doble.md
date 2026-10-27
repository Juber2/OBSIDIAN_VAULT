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
  - ledger, partida doble, debe, haber, asiento, apunte, append-only, cuadre
---

## 🧠 Concepto

La **partida doble** es una técnica contable del siglo XV que sigue siendo la base de todo sistema financiero serio. La idea es simple y potente:

> El dinero no aparece ni desaparece: siempre se mueve **desde** algún lado **hacia** algún lado.

Cada operación genera un **asiento** compuesto por al menos dos **apuntes**, y la suma de los apuntes de un asiento es cero. Como consecuencia, la suma de *todos* los apuntes del sistema también es cero — y esa es una invariante que se puede verificar automáticamente.

## 🎯 Por qué importa en software

Un modelo ingenuo guarda el saldo como una columna:

```sql
UPDATE cuenta SET saldo = saldo - 100 WHERE id = 'A';
UPDATE cuenta SET saldo = saldo + 100 WHERE id = 'B';
```

Problemas, todos graves:

| Problema | Consecuencia |
|---|---|
| No hay historia | ¿por qué el saldo es 1.437,20? nadie puede reconstruirlo |
| Un `UPDATE` mal hecho es indetectable | el dinero se crea o se destruye sin rastro |
| No hay forma de verificar | no existe invariante que se pueda comprobar |
| Concurrencia | dos updates simultáneos y el saldo queda mal |
| Auditoría | imposible demostrar qué pasó |

Con partida doble, **el saldo es un resultado, no un dato**. Y como cada movimiento es una fila nueva que nunca se modifica, la historia completa siempre está disponible.

## 📖 Debe y haber

La contabilidad clásica usa dos columnas en vez de signos:

| | Débito (debe) | Crédito (haber) |
|---|---|---|
| Activo (caja, préstamos) | aumenta | disminuye |
| Pasivo (depósitos de clientes) | disminuye | aumenta |
| Patrimonio | disminuye | aumenta |
| Ingreso | disminuye | aumenta |
| Gasto | aumenta | disminuye |

> [!warning] La inversión que confunde a todo el mundo
> Cuando tu banco te "acredita" S/ 100, para vos es dinero que entra. Para el banco es un **pasivo que aumenta**: te debe S/ 100 más. Tu depósito es deuda del banco con vos.
> Por eso el extracto dice "crédito" cuando te entra plata: está escrito desde la contabilidad del banco, no desde la tuya.

### La simplificación con signo

En software es habitual reemplazar las dos columnas por un importe con signo:

```
débito  → importe negativo
crédito → importe positivo
```

Es equivalente y mucho más simple de programar: verificar que un asiento cuadre es sumar y comparar con cero, en vez de sumar dos columnas por separado.

```java
public record Apunte(CuentaId cuenta, Dinero importe) {
    public Apunte {
        if (importe.esCero()) throw new IllegalArgumentException("apunte en cero");
    }
    public boolean esDebito()  { return importe.esNegativo(); }
    public boolean esCredito() { return importe.esPositivo(); }
}
```

> [!tip] Cuándo conviene volver a las dos columnas
> Si el sistema tiene que integrarse con un ERP contable o emitir reportes regulatorios, probablemente te pidan debe/haber explícitos. Se puede derivar del signo al exportar — pero conviene saber que la conversión existe.

## 🧱 El modelo

```
Asiento  ── id, fecha, motivo, referencia
   │
   └── Apunte[]  ── cuenta, importe (con signo)
        ▲
        └── invariante: la suma por moneda == 0
```

```java
public record Asiento(UUID id, Instant fecha, String motivo, List<Apunte> apuntes) {

    public Asiento {
        if (apuntes == null || apuntes.size() < 2)
            throw new AsientoInvalidoException("se requieren al menos 2 apuntes");
        apuntes = List.copyOf(apuntes);
        exigirQueCuadre(apuntes);
    }

    private static void exigirQueCuadre(List<Apunte> apuntes) {
        Map<Moneda, BigDecimal> porMoneda = apuntes.stream()
            .collect(Collectors.groupingBy(a -> a.importe().moneda(),
                     Collectors.reducing(BigDecimal.ZERO,
                                         a -> a.importe().monto(), BigDecimal::add)));

        porMoneda.forEach((moneda, suma) -> {
            if (suma.signum() != 0)
                throw new AsientoInvalidoException("descuadre de " + suma + " en " + moneda);
        });
    }
}
```

**El cuadre se verifica por moneda, no globalmente** — ver [[02 - Multimoneda y Tipos de Cambio]] para por qué.

> [!tip] La invariante vive en el constructor
> No es una validación que alguien tiene que acordarse de llamar: si el objeto `Asiento` existe, cuadra. Es imposible construir uno descuadrado. Esa es la diferencia entre validar y modelar.

## 💡 Las operaciones, en asientos

Toda operación necesita una **contrapartida**. Nada entra ni sale del sistema sin origen.

```
DEPÓSITO de S/ 500 en la cuenta del cliente
  caja del banco          -500.00
  cuenta cliente          +500.00

RETIRO de S/ 200
  cuenta cliente          -200.00
  caja del banco          +200.00

TRANSFERENCIA de S/ 300 entre clientes
  cuenta origen           -300.00
  cuenta destino          +300.00

COMISIÓN de S/ 5
  cuenta cliente            -5.00
  ingresos por comisiones   +5.00

TRANSFERENCIA con comisión (un solo asiento, 3 apuntes)
  cuenta origen           -305.00
  cuenta destino          +300.00
  ingresos por comisiones   +5.00
```

> [!tip] Un asiento puede tener más de dos apuntes
> Es lo correcto cuando forman parte de **una misma operación atómica**. La transferencia con comisión es un solo hecho económico: partirlo en dos asientos permite que uno quede sin el otro.

## 📊 El plan de cuentas

No todas las cuentas son de clientes. Un ledger real tiene cuentas internas:

| Tipo | Ejemplos | Naturaleza |
|---|---|---|
| Clientes | cuentas de ahorro, corrientes | pasivo |
| Caja y bancos | efectivo, corresponsales | activo |
| Puente / tránsito | operaciones en curso, posición de cambio | transitoria |
| Ingresos | comisiones, intereses ganados | ingreso |
| Egresos | intereses pagados, pérdidas | gasto |
| Ajustes | diferencias de redondeo | técnica |

> [!warning] Las cuentas puente deben quedar en cero
> Una cuenta de tránsito acumula operaciones a medio camino. Si al cierre no está en cero, hay operaciones colgadas — típicamente SAGAs que no completaron ni compensaron. Es una de las alertas más útiles del cierre diario.

## 🗄️ El esquema

```sql
CREATE TABLE asiento (
    id           UUID PRIMARY KEY,
    fecha        TIMESTAMPTZ  NOT NULL,
    motivo       VARCHAR(200) NOT NULL,
    referencia   VARCHAR(60),              -- id externo, para conciliar
    reversa_de   UUID REFERENCES asiento(id),
    creado_por   VARCHAR(60)  NOT NULL
);

CREATE TABLE apunte (
    id           BIGSERIAL PRIMARY KEY,
    asiento_id   UUID          NOT NULL REFERENCES asiento(id),
    cuenta_id    UUID          NOT NULL REFERENCES cuenta(id),
    monto        NUMERIC(19,4) NOT NULL CHECK (monto <> 0),
    moneda       VARCHAR(3)    NOT NULL
);

CREATE INDEX idx_apunte_cuenta  ON apunte(cuenta_id, id DESC);
CREATE INDEX idx_asiento_fecha  ON asiento(fecha DESC);
CREATE UNIQUE INDEX idx_asiento_referencia
    ON asiento(referencia) WHERE referencia IS NOT NULL;
```

### Append-only, impuesto por la base

```sql
REVOKE UPDATE, DELETE ON asiento, apunte FROM corebank_app;
```

> [!danger] Nunca se edita ni se borra un asiento
> Ni para corregir un error, ni para "limpiar datos de prueba", ni porque lo pidió un gerente. Un error se corrige con un **asiento de reversa** que deja los dos hechos registrados: el error y su corrección.
> Un ledger que admite `UPDATE` no es un ledger: es una tabla.

## ↩️ La reversa

```java
public Asiento reversar(Asiento original, String motivo, String usuario) {
    List<Apunte> invertidos = original.apuntes().stream()
        .map(ap -> new Apunte(ap.cuenta(), ap.importe().negado()))
        .toList();

    return new Asiento(UUID.randomUUID(), Instant.now(),
        "REVERSA[%s] %s".formatted(original.id(), motivo), invertidos);
}
```

Después de reversar, el saldo vuelve a su valor previo **y existen dos asientos**. El extracto muestra los dos, que es exactamente lo que el cliente y el auditor necesitan ver.

## 🧮 Calcular el saldo

```sql
-- Saldo actual
SELECT COALESCE(SUM(monto), 0)
FROM apunte
WHERE cuenta_id = :cuenta AND moneda = :moneda;

-- Saldo a una fecha pasada: sale gratis
SELECT COALESCE(SUM(a.monto), 0)
FROM apunte a JOIN asiento s ON s.id = a.asiento_id
WHERE a.cuenta_id = :cuenta AND a.moneda = :moneda AND s.fecha <= :corte;
```

> [!tip] El saldo histórico es la ventaja gratis del ledger
> Con una columna `saldo` no podés responder "¿cuánto tenía este cliente el 3 de marzo?". Con un ledger es un `WHERE`. Los reportes regulatorios lo piden constantemente.

Cuando el volumen lo exige, se agrega un **snapshot** — sin dejar de derivar la verdad de los apuntes. Ver [[Fase 8 - Batch y Cierre de Dia]].

## ✅ El cuadre global

```sql
SELECT moneda, SUM(monto) AS descuadre
FROM apunte
GROUP BY moneda
HAVING SUM(monto) <> 0;
```

Si devuelve filas, hay un bug **grave**: se creó o destruyó dinero. Este control se corre en cada cierre y detiene el proceso si falla.

```java
@Test
void tras_cualquier_secuencia_de_operaciones_el_sistema_cuadra() {
    // property-based: N operaciones aleatorias, el total siempre da cero
}
```

## ⚠️ Errores Comunes

> [!warning] Los que rompen un ledger
> - **Guardar el saldo y actualizarlo.** Dos fuentes de verdad que van a divergir.
> - **Un solo apunte por operación.** El dinero aparece de la nada; el cuadre global nunca da cero.
> - **Editar o borrar asientos.** Destruye la auditoría y hace irreconstruible el saldo.
> - **Cuadrar globalmente en multi-moneda.** Da cero por casualidad o nunca.
> - **No modelar las cuentas internas.** Sin caja ni ingresos, no hay dónde poner la contrapartida.
> - **Reversar con `DELETE`.** Es el mismo error que editar, con otro nombre.

## 📌 Notas Rápidas
- La partida doble tiene 500 años y sigue vigente porque es un **sistema de detección de errores**: si te equivocás en un apunte, el asiento no cuadra y el error salta de inmediato.
- Sistemas como TigerBeetle o Ledger de Square existen específicamente para esto: bases de datos cuyo modelo primitivo es el asiento contable. Vale conocer que existen.
- La cuenta de "diferencias de redondeo" no es opcional en multi-moneda: los céntimos que genera cada conversión tienen que ir a algún lado, o son tu descuadre.
- Se implementa en [[Fase 0 - Ledger en Java Puro]] y se verifica en [[Fase 8 - Batch y Cierre de Dia]].

## 🔗 Relaciones
- **Módulo**: [[Banca y Fintech]]
- **Anterior**: [[02 - Multimoneda y Tipos de Cambio]]
- **Siguiente**: [[04 - Cuentas Saldos y Movimientos]]
- **Se conecta con**: [[09 - Conciliacion y Cuadre]], [[12 - Auditoria y Trazabilidad]], [[08 - Event Sourcing]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

