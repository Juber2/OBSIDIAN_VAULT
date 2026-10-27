---
tags:
  - Java
  - Banca
  - Pagos
Creado: 2026-09-08
Relacionado:
  - Banca y Fintech
  - 04 - Cuentas Saldos y Movimientos
Palabras clave:
  - ISO 8583, tarjetas, autorizacion, clearing, settlement, MTI, STAN, RRN, adquirente, emisor
---

## 🧠 Concepto

Un pago con tarjeta no es una operación: son **tres fases separadas en el tiempo**, con actores distintos y mensajes distintos. Confundirlas es la fuente de la mayoría de los errores al integrar con medios de pago.

## 👥 Los actores

```mermaid
flowchart LR
    TH["Tarjetahabiente"] --> COM["Comercio"]
    COM --> ADQ["Adquirente<br/>banco del comercio"]
    ADQ --> RED{{"Red de marca<br/>Visa · Mastercard"}}
    RED --> EMI["Emisor<br/>banco del cliente"]
    EMI --> RED
    RED --> ADQ
    ADQ --> COM

    style RED fill:#1d3557,color:#fff
    style EMI fill:#6a040f,color:#fff
```

| Actor | Rol |
|---|---|
| **Tarjetahabiente** | quien paga |
| **Comercio** | quien cobra |
| **Adquirente** | el banco del comercio; procesa el cobro |
| **Red de marca** | rutea el mensaje entre adquirente y emisor |
| **Emisor** | el banco que emitió la tarjeta; **autoriza o rechaza** |
| **Procesador** | opera la infraestructura por cuenta de alguno de los anteriores |

> [!tip] Quién decide es siempre el emisor
> El comercio no aprueba nada, y el adquirente tampoco. La decisión de aprobar o rechazar la toma el banco que emitió la tarjeta, que es el único que conoce el saldo y los límites del cliente.

## 🔄 Las tres fases

| Fase | Cuándo | Qué pasa | ¿Mueve dinero? |
|---|---|---|---|
| **Autorización** | al momento de pagar | el emisor valida y **retiene** fondos | ❌ solo retención |
| **Clearing** | horas o días después | el comercio presenta la operación para cobro | ❌ intercambio de archivos |
| **Settlement** | según ciclo | se liquidan los fondos entre entidades | ✅ |

```
Día 0 · 14:32  autorización aprobada  → retención de S/ 200 (ver [[04 - Cuentas Saldos y Movimientos]])
Día 1 · 03:00  clearing               → el adquirente presenta el lote
Día 2         settlement              → asiento real de -S/ 200, se libera la retención
```

> [!danger] La autorización no es un asiento contable
> Es una **retención**. Si generás el asiento en la autorización, después llega la liquidación y lo duplicás. Y si la operación nunca se presenta a clearing, registraste un movimiento que jamás ocurrió.

### Casos que rompen el flujo feliz

| Caso | Qué pasa |
|---|---|
| Autoriza y nunca se presenta | la retención caduca y se libera |
| Se presenta por menos | propinas, ajustes: se liquida el monto real |
| Se presenta por más | permitido en rubros como hotelería, con tolerancia |
| Anulación (void) | antes del clearing: se cancela sin asiento |
| Reversa | después: asiento inverso |
| Contracargo (chargeback) | el cliente desconoce el cargo; disputa formal |

## 📨 ISO 8583

El estándar de mensajería para transacciones con tarjeta. Es **binario y posicional**, no XML ni JSON — diseñado en los 80 para líneas lentas, y sigue vigente en toda la infraestructura de tarjetas.

Un mensaje tiene tres partes:

```
┌─────────┬──────────────────┬───────────────────────────┐
│   MTI   │     Bitmap(s)    │      Data Elements        │
│ 4 díg.  │  64 o 128 bits   │  campos presentes         │
└─────────┴──────────────────┴───────────────────────────┘
```

### MTI — Message Type Indicator

Cuatro dígitos que dicen qué es el mensaje:

| MTI | Significado |
|---|---|
| `0100` | solicitud de autorización |
| `0110` | respuesta de autorización |
| `0200` | solicitud financiera (autoriza y captura) |
| `0210` | respuesta financiera |
| `0220` | aviso financiero |
| `0400` | solicitud de reversa |
| `0410` | respuesta de reversa |
| `0420` | aviso de reversa |
| `0800` | gestión de red (echo, sign-on) |
| `0810` | respuesta de gestión de red |

### Bitmap

Un mapa de bits que indica **qué campos vienen** en el mensaje. El bitmap primario cubre los campos 1-64; si el bit 1 está encendido, hay un bitmap secundario para los campos 65-128.

```
Bitmap: 0111001000000000...
        ││││  │
        ││││  └── bit 6 encendido → viene el DE 6
        │││└───── bit 4 encendido → viene el DE 4 (monto)
        ││└────── bit 3 encendido → viene el DE 3
        │└─────── bit 2 encendido → viene el DE 2 (PAN)
        └──────── bit 1 apagado   → no hay bitmap secundario
```

Es lo que hace el formato compacto: solo se transmiten los campos presentes.

### Data Elements que hay que conocer

| DE | Nombre | Contenido |
|---|---|---|
| 2 | PAN | número de tarjeta |
| 3 | Processing code | tipo de operación y cuentas involucradas |
| 4 | Monto | **12 dígitos sin punto decimal** |
| 7 | Fecha/hora de transmisión | MMDDhhmmss |
| 11 | **STAN** | número de rastreo, único por día y terminal |
| 12 / 13 | Hora y fecha local | |
| 22 | POS entry mode | banda, chip, contactless, manual |
| 35 | Track 2 | datos de la banda |
| 37 | **RRN** | referencia de recuperación, 12 caracteres |
| 38 | Código de aprobación | 6 caracteres, si aprobó |
| 39 | **Response code** | resultado |
| 41 | Terminal ID | |
| 42 | Merchant ID | |
| 49 | Código de moneda | ISO 4217 **numérico** (`604` = PEN) |
| 52 | PIN block | cifrado |
| 55 | Datos EMV | chip |

> [!danger] El DE 4 no lleva punto decimal
> `000000015000` con moneda `604` (PEN, 2 decimales) significa **S/ 150,00**. El separador está implícito en el exponente de la moneda.
> Si asumís 2 decimales fijos, con yenes (0 decimales) vas a cobrar 100 veces menos.

```java
public Dinero parsearMonto(String de4, String de49) {
    Moneda moneda = Moneda.porCodigoNumerico(de49);       // 604 → PEN
    BigDecimal crudo = new BigDecimal(de4);               // 000000015000 → 15000
    return new Dinero(
        crudo.movePointLeft(moneda.decimales()), moneda); // → 150.00
}
```

### Códigos de respuesta (DE 39)

| Código | Significado | ¿Reintentar? |
|---|---|---|
| `00` | aprobada | — |
| `01` | referir al emisor | ❌ |
| `05` | no honrar (rechazo genérico) | ❌ |
| `12` | transacción inválida | ❌ |
| `14` | número de tarjeta inválido | ❌ |
| `41` / `43` | tarjeta perdida / robada | ❌ **retener** |
| `51` | fondos insuficientes | ❌ |
| `54` | tarjeta vencida | ❌ |
| `55` | PIN incorrecto | ❌ |
| `61` | excede límite de retiro | ❌ |
| `91` | emisor no disponible | ✅ transitorio |
| `96` | error del sistema | ✅ transitorio |

> [!warning] Solo `91` y `96` se reintentan
> Los demás son rechazos deterministas: reintentar da el mismo resultado y solo agrega carga. Peor aún, un reintento sobre un `05` puede disparar reglas antifraude del emisor.

## 🔑 Identificadores y trazabilidad

Tres identificadores distintos que se confunden:

| Campo | Ámbito | Para qué |
|---|---|---|
| **STAN** (DE 11) | terminal + día | rastrear dentro de la jornada; **se recicla** |
| **RRN** (DE 37) | adquirente | referencia de la operación; para conciliar |
| **Código de aprobación** (DE 38) | emisor | prueba de que autorizó |

> [!danger] El STAN no es único de forma global
> Son 6 dígitos que se reciclan a diario y por terminal. Usarlo como clave primaria produce colisiones. Para identificar unívocamente una operación se combina RRN + adquirente + fecha, o se genera un identificador propio.

## ⏪ La reversa (0400)

Si el adquirente no recibe respuesta a una autorización, **no sabe si el emisor la aprobó**. Es el problema del estado desconocido de [[06 - Transferencias y Consistencia]], en el mundo de tarjetas.

El protocolo lo resuelve con la reversa: el adquirente envía un `0400` que dice "cancelá esa autorización, no la usé". El emisor libera la retención.

```java
public void manejarTimeoutDeAutorizacion(SolicitudAutorizacion solicitud) {
    // NO se asume que fue rechazada: pudo haberse aprobado
    enviarReversa(Mensaje8583.builder()
        .mti("0400")
        .de(11, solicitud.stan())        // mismos identificadores
        .de(37, solicitud.rrn())
        .de(4,  solicitud.montoFormateado())
        .build());
}
```

> [!warning] Las reversas se reintentan hasta confirmar
> A diferencia de las autorizaciones, una reversa **debe** llegar. Si no se confirma, la retención queda viva y el cliente tiene fondos bloqueados sin motivo. Los switches reintentan reversas durante horas.

## 🔐 Datos sensibles

Del lado de PCI-DSS (detalle en [[11 - PCI-DSS y HSM]]), lo mínimo a saber acá:

| Dato | ¿Se puede almacenar? |
|---|---|
| PAN | sí, **ilegible**: cifrado, truncado o tokenizado |
| Nombre, vencimiento | sí, con protección |
| CVV/CVC | ❌ **nunca**, ni cifrado |
| Track completo (banda) | ❌ **nunca** tras la autorización |
| PIN / PIN block | ❌ **nunca** |

```java
public record PanEnmascarado(String bin, String ultimos4) {
    public static PanEnmascarado de(String pan) {
        return new PanEnmascarado(pan.substring(0, 6), pan.substring(pan.length() - 4));
    }
    @Override public String toString() { return bin + "******" + ultimos4; }
}
```

> [!danger] Un PAN completo en un log es un incidente de seguridad
> No un bug: un incidente reportable. Los logs se exportan, se indexan en servicios de terceros y se respaldan. Enmascará en el propio serializador, nunca confiando en que cada desarrollador se acuerde.

## 🧪 Tests

```java
@Test
void el_monto_se_interpreta_segun_el_exponente_de_la_moneda() {
    assertEquals(Dinero.de("150.00", PEN), parsearMonto("000000015000", "604"));
    assertEquals(Dinero.de("15000",  JPY), parsearMonto("000000015000", "392"));
}

@Test
void un_rechazo_determinista_no_se_reintenta() {
    assertFalse(politica.debeReintentar("51"));
    assertTrue(politica.debeReintentar("91"));
}

@Test
void un_timeout_de_autorizacion_dispara_reversa() {
    when(switch_.autorizar(any())).thenThrow(new TimeoutException());
    servicio.autorizar(solicitud);
    verify(switch_).enviarReversa(argThat(m -> m.mti().equals("0400")));
}

@Test
void ningun_log_contiene_el_pan_completo() {
    // escanea la salida de log buscando patrones de 13-19 dígitos
}
```

## ⚠️ Errores Comunes

> [!warning] Los que rompen una integración
> - **Generar el asiento en la autorización.** Se duplica al liquidar.
> - **Asumir 2 decimales** en el DE 4 ignorando el exponente de la moneda.
> - **Usar el STAN como identificador único.** Se recicla a diario.
> - **Reintentar rechazos deterministas.** Carga inútil y riesgo antifraude.
> - **No enviar reversa ante timeout.** El cliente queda con fondos retenidos.
> - **Almacenar CVV o track completo.** Prohibido, sin excepciones.
> - **Loguear el PAN.** Incidente de seguridad.

## 📌 Notas Rápidas
- ISO 8583 tiene tres versiones (1987, 1993, 2003) y cada red define su propio dialecto: los campos privados (DE 60-63, 120-127) los usa cada marca a su manera. **Siempre hay que leer la especificación del procesador concreto**, no solo el estándar.
- `jPOS` es la biblioteca de referencia en Java para ISO 8583. Es vieja y con documentación escasa, pero es lo que se usa en producción en toda la industria.
- Las tres fases (autorización, clearing, settlement) explican por qué un cargo aparece "pendiente" en la app y días después cambia de monto o desaparece: no es un error, es el ciclo normal.
- Se relaciona con las retenciones de [[04 - Cuentas Saldos y Movimientos]] y con la conciliación de [[09 - Conciliacion y Cuadre]].

## 🔗 Relaciones
- **Módulo**: [[Banca y Fintech]]
- **Anterior**: [[06 - Transferencias y Consistencia]]
- **Siguiente**: [[08 - ISO 20022 y SWIFT]]
- **Se conecta con**: [[11 - PCI-DSS y HSM]], [[13 - Antifraude y Limites]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

