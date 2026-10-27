---
tags:
  - Java
  - Banca
  - Pagos
Creado: 2026-09-08
Relacionado:
  - Banca y Fintech
  - 07 - Medios de Pago y ISO 8583
Palabras clave:
  - ISO 20022, SWIFT, pacs, pain, camt, MT103, BIC, IBAN, UETR, nostro, vostro
---

## 🧠 Concepto

Si ISO 8583 es el lenguaje de las tarjetas, **ISO 20022 es el de las transferencias** — entre bancos, entre países y cada vez más también dentro de los sistemas nacionales de pagos.

Es XML, está tipado, es autodescriptivo y trae mucha más información que los formatos que reemplaza. La industria está migrando a él desde los mensajes SWIFT MT.

## 📛 Cómo se nombran los mensajes

```
pacs . 008 . 001 . 08
 │      │     │     │
 │      │     │     └── versión
 │      │     └──────── variante
 │      └────────────── identificador del mensaje
 └───────────────────── área de negocio
```

### Áreas de negocio

| Prefijo | Área | Ejemplo de uso |
|---|---|---|
| `pain` | Payments **In**itiation | el cliente instruye a su banco |
| `pacs` | Payments **C**learing and **S**ettlement | banco → banco |
| `camt` | **Ca**sh **M**anagemen**t** | extractos y notificaciones |
| `acmt` | **Acc**oun**t** Management | apertura y gestión de cuentas |
| `remt` | **Rem**ittance | información de remesa |
| `head` | Business Application Header | cabecera común |

### Los mensajes que hay que conocer

| Mensaje | Qué es | Dirección |
|---|---|---|
| `pain.001` | iniciación de transferencia | cliente → su banco |
| `pain.002` | estado de la instrucción | banco → cliente |
| `pacs.008` | transferencia de cliente | banco → banco |
| `pacs.009` | transferencia entre entidades | banco → banco (fondos propios) |
| `pacs.002` | reporte de estado del pago | banco → banco |
| `pacs.004` | devolución de pago | banco → banco |
| `camt.052` | reporte intradiario | banco → cliente |
| `camt.053` | **extracto de fin de día** | banco → cliente |
| `camt.054` | notificación de débito/crédito | banco → cliente |
| `camt.056` | solicitud de cancelación | banco → banco |

> [!tip] El flujo completo de una transferencia
> `pain.001` (el cliente ordena) → `pacs.008` (su banco lo envía al banco destino) → `pacs.002` (el destino confirma) → `camt.054` (se notifica el crédito) → `camt.053` (aparece en el extracto del día).

## 🔄 Migración desde SWIFT MT

Los mensajes MT son el formato clásico de SWIFT: texto plano con campos numerados. Están siendo reemplazados por ISO 20022 (llamados MX en ese contexto).

| MT (clásico) | ISO 20022 | Qué es |
|---|---|---|
| MT103 | `pacs.008` | transferencia de cliente |
| MT202 | `pacs.009` | transferencia entre bancos |
| MT940 | `camt.053` | extracto de fin de día |
| MT942 | `camt.052` | reporte intradiario |
| MT900/910 | `camt.054` | confirmación de débito/crédito |

```
:20:REFERENCIA123          ← MT103: campos posicionales, texto plano
:23B:CRED
:32A:260908PEN1500,50      ← fecha + moneda + monto (coma decimal)
:50K:/ACC-001
JUBER GUTIERREZ
:59:/ACC-002
MARIA LOPEZ
:71A:SHA
```

> [!warning] El monto MT usa coma decimal, no punto
> `PEN1500,50` son mil quinientos con cincuenta. Parsear con `Double.parseDouble` falla o —peor— interpreta mal según el locale. Es uno de los errores clásicos al integrar formatos MT.

**Por qué se migra:** los campos MT son cortos y poco estructurados, lo que obliga a meter información en campos de texto libre. ISO 20022 permite datos estructurados de ordenante, beneficiario, propósito del pago y remesa — que es lo que necesitan las áreas de cumplimiento y antilavado.

## 📄 Anatomía de un `pacs.008`

```xml
<Document xmlns="urn:iso:std:iso:20022:tech:xsd:pacs.008.001.08">
  <FIToFICstmrCdtTrf>
    <GrpHdr>
      <MsgId>MSG-20260908-000123</MsgId>
      <CreDtTm>2026-09-08T14:32:11Z</CreDtTm>
      <NbOfTxs>1</NbOfTxs>
      <SttlmInf><SttlmMtd>INDA</SttlmMtd></SttlmInf>
    </GrpHdr>
    <CdtTrfTxInf>
      <PmtId>
        <EndToEndId>FACTURA-2026-4471</EndToEndId>
        <TxId>TX-000123</TxId>
        <UETR>7f3a9b2c-4d5e-4f70-8192-a3b4c5d6e7f8</UETR>
      </PmtId>
      <IntrBkSttlmAmt Ccy="PEN">1500.50</IntrBkSttlmAmt>
      <IntrBkSttlmDt>2026-09-08</IntrBkSttlmDt>
      <Dbtr><Nm>Juber Gutierrez</Nm></Dbtr>
      <DbtrAcct><Id><Othr><Id>00212345678901234567</Id></Othr></Id></DbtrAcct>
      <DbtrAgt><FinInstnId><BICFI>BCPXPEPL</BICFI></FinInstnId></DbtrAgt>
      <CdtrAgt><FinInstnId><BICFI>BINPPEPL</BICFI></FinInstnId></CdtrAgt>
      <Cdtr><Nm>Maria Lopez</Nm></Cdtr>
      <CdtrAcct><Id><Othr><Id>00398765432109876543</Id></Othr></Id></CdtrAcct>
      <RmtInf><Ustrd>Pago factura 4471</Ustrd></RmtInf>
    </CdtTrfTxInf>
  </FIToFICstmrCdtTrf>
</Document>
```

> [!tip] El monto sí lleva punto decimal y atributo de moneda
> `<IntrBkSttlmAmt Ccy="PEN">1500.50</IntrBkSttlmAmt>` — a diferencia de ISO 8583, acá el decimal es explícito y la moneda va como atributo. Se mapea directo a `BigDecimal`.

### Los tres identificadores

| Campo | Alcance | Uso |
|---|---|---|
| `MsgId` | el mensaje | control de transmisión |
| `EndToEndId` | asignado por el ordenante | **viaja intacto** hasta el beneficiario; para conciliar |
| `UETR` | UUID v4, único global | rastreo punta a punta |

> [!tip] El `EndToEndId` es tu clave de conciliación
> Es el único campo que el estándar garantiza que llega sin modificar al beneficiario. Si ponés ahí el identificador de tu factura o de tu operación, después podés cruzar automáticamente lo que vuelve — ver [[09 - Conciliacion y Cuadre]].

El **UETR** (Unique End-to-end Transaction Reference) es un UUID v4 obligatorio en SWIFT desde 2018. Es lo que permite el rastreo tipo *"seguimiento de encomienda"* de las transferencias internacionales.

## 🏦 BIC e IBAN

### BIC (código SWIFT)

```
BCPXPEPL XXX
│││││││└─────── sucursal (opcional, 3 caracteres)
││││││└──────── ubicación (2)
││││└────────── país ISO 3166 (2)
└────────────── código de entidad (4)
```

8 u 11 caracteres. Identifica la institución financiera.

### IBAN

```
ES91 2100 0418 4502 0005 1332
││ │ └──────────────────────── BBAN: identificación nacional de la cuenta
││ └────────────────────────── dígitos de control (mod-97)
└└──────────────────────────── país ISO 3166
```

Longitud total variable por país (hasta 34 caracteres), pero fija dentro de cada uno.

```java
public static boolean ibanValido(String iban) {
    String limpio = iban.replaceAll("\\s", "").toUpperCase();
    if (limpio.length() < 15 || limpio.length() > 34) return false;

    // 1 · Mover los 4 primeros caracteres al final
    String reordenado = limpio.substring(4) + limpio.substring(0, 4);

    // 2 · Cada letra → su posición + 9  (A=10 ... Z=35); los dígitos quedan igual
    StringBuilder numerico = new StringBuilder();
    for (char c : reordenado.toCharArray()) {
        if (Character.isDigit(c)) {
            numerico.append(c);
        } else if (c >= 'A' && c <= 'Z') {
            numerico.append(c - 'A' + 10);      // se agregan DOS dígitos
        } else {
            return false;                        // carácter no permitido
        }
    }

    // 3 · Es válido si el número resultante módulo 97 da 1
    return new BigInteger(numerico.toString())
        .mod(BigInteger.valueOf(97))
        .equals(BigInteger.ONE);
}
```

> [!tip] Validar el IBAN antes de enviar ahorra dinero real
> Una transferencia internacional rechazada por IBAN inválido cuesta comisiones de devolución y días de demora. La validación mod-97 es local, instantánea y detecta la mayoría de los errores de tipeo.

**Perú no usa IBAN**: el identificador interbancario es el **CCI**, de 20 dígitos (entidad + oficina + cuenta + dígitos de control). Cada país tiene el suyo: CLABE en México, CBU en Argentina, chave PIX en Brasil. La idea es la misma — identificador único con dígitos de verificación.

## 💱 Cómo se mueve el dinero: nostro y vostro

> [!danger] SWIFT no transfiere dinero
> Es una red de **mensajería**: transporta instrucciones, no fondos. El dinero se mueve por cuentas de corresponsalía entre bancos.

| Término | Significado | Perspectiva |
|---|---|---|
| **Nostro** | "nuestra cuenta en tu banco" | dinero propio en el exterior |
| **Vostro** | "tu cuenta en nuestro banco" | dinero ajeno en casa |

Cuando el banco A instruye un pago al banco B, en realidad se debita la cuenta nostro de A en B (o se usa un corresponsal intermedio si no tienen relación directa). Por eso las transferencias internacionales pueden pasar por dos o tres bancos y acumular comisiones.

```
Banco A (Perú) ──► Corresponsal (EE.UU.) ──► Banco B (España)
       cada salto: un mensaje, una comisión, un día
```

## 🇵🇪 Sistemas nacionales

Cada país tiene su cámara de compensación para transferencias domésticas, y casi todas están migrando o ya migraron a ISO 20022:

| País | Sistema |
|---|---|
| Perú | CCE (Cámara de Compensación Electrónica) |
| México | SPEI |
| Brasil | PIX |
| Argentina | Transferencias 3.0 / COELSA |
| Zona euro | SEPA (SCT / SCT Inst) |
| EE.UU. | FedNow, RTP, ACH |

> [!warning] Cada sistema define su propio perfil de uso
> ISO 20022 es un estándar amplio: cada implementación restringe qué campos son obligatorios, con qué longitudes y qué códigos admite. SWIFT publica **CBPR+** para pagos transfronterizos; SEPA tiene el suyo. **Siempre hay que leer el perfil concreto**, no solo el esquema base.

## 💻 Trabajar con esto en Java

```java
// Generación de clases desde el XSD oficial con JAXB
// mvn: jaxb2-maven-plugin, generando desde pacs.008.001.08.xsd

var contexto = JAXBContext.newInstance(Document.class);

// La validación contra el XSD NO es opcional
var esquema = SchemaFactory.newInstance(W3C_XML_SCHEMA_NS_URI)
    .newSchema(getClass().getResource("/xsd/pacs.008.001.08.xsd"));

var unmarshaller = contexto.createUnmarshaller();
unmarshaller.setSchema(esquema);          // ← rechaza mensajes mal formados
Document doc = (Document) unmarshaller.unmarshal(entrada);
```

> [!danger] Configurá el parser XML contra XXE
> Un XML de un tercero puede incluir entidades externas que lean archivos del servidor. Es una vulnerabilidad del OWASP Top 10.
> ```java
> var factory = SAXParserFactory.newInstance();
> factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
> factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
> ```

## 🧪 Tests

```java
@Test
void un_mensaje_valido_pasa_la_validacion_del_esquema() { ... }

@Test
void un_mensaje_con_campo_obligatorio_faltante_es_rechazado() { ... }

@Test
void el_endToEndId_sobrevive_el_viaje_de_ida_y_vuelta() {
    var enviado = construirPacs008(referencia("FACTURA-4471"));
    var respuesta = parsearPacs002(simularRespuesta(enviado));
    assertEquals("FACTURA-4471", respuesta.endToEndId());
}

@Test
void el_validador_de_iban_detecta_errores_de_tipeo() {
    assertTrue(ibanValido("ES91 2100 0418 4502 0005 1332"));
    assertFalse(ibanValido("ES91 2100 0418 4502 0005 1333"));   // un dígito cambiado
}

@Test
void el_parser_rechaza_entidades_externas() { /* XXE */ }
```

## ⚠️ Errores Comunes

> [!warning] Los que se pagan caro
> - **Parsear montos MT con `Double`**: coma decimal + locale = desastre.
> - **No validar contra el XSD**: los errores aparecen en el banco destino, no en el tuyo.
> - **Usar el `MsgId` para conciliar** en vez del `EndToEndId`: solo el segundo viaja intacto.
> - **No validar IBAN/CCI antes de enviar**: devoluciones con comisión y demora.
> - **Ignorar el perfil de uso** del sistema concreto y quedarse con el esquema base.
> - **Parser XML sin protección XXE**.
> - **Creer que SWIFT mueve dinero**: mueve mensajes; el dinero va por nostro/vostro.

## 📌 Notas Rápidas
- La migración global de MT a ISO 20022 es un proyecto plurianual que atraviesa toda la banca: es una fuente sostenida de trabajo para backends. Mencionarla con propiedad en una entrevista de banca posiciona bien.
- ISO 20022 es verboso a propósito: prioriza que los datos estén estructurados sobre que el mensaje sea compacto. Es lo opuesto a ISO 8583, y responde a que hoy el ancho de banda es barato y el cumplimiento normativo es caro.
- El `EndToEndId` es el equivalente conceptual de una clave de idempotencia entre entidades — ver [[05 - Idempotencia Financiera]].
- `Prowide Core` es una biblioteca Java de código abierto para mensajes SWIFT MT y MX; es lo más usado si hay que trabajar con estos formatos.

## 🔗 Relaciones
- **Módulo**: [[Banca y Fintech]]
- **Anterior**: [[07 - Medios de Pago y ISO 8583]]
- **Siguiente**: [[09 - Conciliacion y Cuadre]]
- **Se conecta con**: [[14 - Open Banking]], [[06 - Transferencias y Consistencia]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

