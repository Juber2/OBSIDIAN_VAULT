---
tags:
  - Java
  - Banca
  - Seguridad
Creado: 2026-09-08
Relacionado:
  - Banca y Fintech
  - 07 - Medios de Pago y ISO 8583
Palabras clave:
  - PCI-DSS, HSM, CHD, SAD, tokenizacion, PIN block, DUKPT, cifrado, alcance
---

## 🧠 Concepto

**PCI-DSS** (Payment Card Industry Data Security Standard) es el estándar que define cómo hay que proteger los datos de tarjetas. No es una ley: es un requisito contractual de las marcas (Visa, Mastercard, Amex). Incumplirlo cuesta multas, aumento de comisiones y, en el extremo, perder la capacidad de procesar tarjetas.

Para un desarrollador, casi todo se reduce a dos preguntas: **qué dato puedo guardar** y **cómo reduzco lo que toco**.

## 🗂️ Qué se puede almacenar

La distinción más importante del estándar:

### CHD — Cardholder Data

| Dato | ¿Almacenable? | Condición |
|---|---|---|
| PAN (número de tarjeta) | ✅ | **ilegible**: cifrado, truncado, hasheado o tokenizado |
| Nombre del titular | ✅ | protegido |
| Fecha de vencimiento | ✅ | protegido |
| Código de servicio | ✅ | protegido |

### SAD — Sensitive Authentication Data

| Dato | ¿Almacenable? |
|---|---|
| Track completo de la banda | ❌ **nunca** |
| CVV / CVC / CID | ❌ **nunca** |
| PIN / PIN block | ❌ **nunca** |

> [!danger] SAD no se almacena tras la autorización. Ni cifrado.
> Es la regla más estricta y la más violada. "Lo guardamos cifrado" **no** es una excepción válida: el estándar prohíbe la persistencia, no la persistencia en claro.
> Y aplica a todo: base de datos, logs, archivos temporales, colas, cachés, respaldos y capturas de pantalla de soporte.

### Enmascarado del PAN

Al mostrarlo, como máximo los **primeros 6 y los últimos 4**:

```java
public record PanEnmascarado(String bin, String ultimos4) {

    public static PanEnmascarado de(String pan) {
        String limpio = pan.replaceAll("[^0-9]", "");
        return new PanEnmascarado(limpio.substring(0, 6),
                                  limpio.substring(limpio.length() - 4));
    }

    @Override public String toString() {
        return bin + "*".repeat(6) + ultimos4;      // 411111******1111
    }
}
```

> [!tip] Los primeros 6 dígitos son el BIN
> Identifican al emisor y el tipo de producto, no al titular. Sirven para enrutamiento y reglas de negocio, y por eso el estándar permite mostrarlos.

## 🎯 Reducir el alcance

El alcance (*scope*) de PCI-DSS es **todo sistema que almacena, procesa o transmite datos de tarjeta**, más todo lo que esté conectado a ellos. Cuanto más chico el alcance, menos costosa la certificación.

| Estrategia | Cómo funciona | Reducción |
|---|---|---|
| **Tokenización** | guardás un token, el PAN vive en un proveedor certificado | muy alta |
| **Redirección / iframe** | el navegador manda el PAN directo al proveedor | máxima |
| **P2PE** | cifrado punto a punto desde el terminal | alta |
| **Segmentación de red** | aísla el entorno de tarjetas | media |

```java
// ❌ El PAN entra a tu sistema: quedás dentro del alcance
public record PagoRequest(String pan, String cvv, String vencimiento) {}

// ✅ Solo recibís un token: el PAN nunca tocó tu servidor
public record PagoRequest(String tokenTarjeta, BigDecimal monto) {}
```

> [!tip] La mejor estrategia es no tocar el dato
> Si el navegador o la app envían el PAN directamente al proveedor de pagos y tu backend solo recibe un token, la mayor parte del alcance desaparece. Es la arquitectura por defecto de Stripe, Adyen y similares — y es una decisión de diseño, no de seguridad "agregada después".

### Tokenización

```
PAN real:  4111 1111 1111 1111
Token:     tok_1a2b3c4d5e6f7g8h        ← esto es lo que guardás
```

| | Cifrado | Tokenización |
|---|---|---|
| Reversible con la clave | ✅ | ❌ (el token no contiene el PAN) |
| Si te roban la base | el atacante necesita la clave | el token no sirve fuera del sistema |
| Reduce el alcance | poco | mucho |

## 🔐 HSM

Un **HSM** (Hardware Security Module) es un dispositivo físico, resistente a manipulación, que guarda claves criptográficas y ejecuta operaciones **sin que la clave salga nunca en claro**.

```
      ┌──────────────────────────┐
      │           HSM            │
      │  ┌────────────────────┐  │
App ──┼─►│ claves (nunca salen)│  │
      │  └────────────────────┘  │
      │   cifra · descifra ·     │
      │   firma · verifica PIN   │
      └──────────┬───────────────┘
                 │
         solo entra y sale el RESULTADO
```

> [!warning] La diferencia con guardar una clave en un archivo
> Si la clave está en el sistema de archivos o en una variable de entorno, cualquiera con acceso al servidor la copia. En un HSM la clave se genera adentro, se usa adentro y nunca se puede extraer. Si alguien abre el aparato, se borra sola.

### Para qué se usa

| Operación | Por qué en HSM |
|---|---|
| Verificación de PIN | el PIN en claro nunca debe existir fuera del hardware |
| Traducción de PIN block | descifrar con una clave y recifrar con otra, atómicamente |
| Generación de claves | entropía certificada |
| Cifrado del PAN | la clave maestra no sale |
| Firma de mensajes | no repudio |
| Generación de CVV | algoritmo con clave secreta |

### La jerarquía de claves

```
LMK  Local Master Key      ← vive solo en el HSM, protege a todas las demás
 ├── ZMK  Zone Master Key   ← intercambio con otra entidad
 │    └── ZPK  Zone PIN Key ← cifra PIN blocks entre zonas
 ├── TMK  Terminal Master Key
 │    └── TPK  Terminal PIN Key
 └── PVK  PIN Verification Key
```

Las claves de trabajo se guardan **cifradas bajo la LMK**. Sin el HSM son bytes inútiles: por eso un respaldo de la base no compromete nada.

### PIN block y traducción

El PIN nunca viaja en claro. Se empaqueta en un **PIN block** (formatos ISO 9564: 0, 1, 3 y 4) que combina el PIN con parte del PAN y se cifra.

```
Terminal ──[PIN cifrado con TPK]──► Adquirente
                                        │
                                    HSM traduce:
                                    descifra con TPK
                                    recifra con ZPK
                                    (el PIN en claro solo existe
                                     dentro del hardware)
                                        │
                                        ▼
                                    Emisor ──► HSM verifica
```

> [!danger] Esto es exactamente por qué existe el HSM
> La traducción de PIN requiere descifrar y recifrar. Si se hiciera en software, habría un instante en que el PIN está en claro en la memoria del servidor — donde un volcado de memoria o un depurador lo capturaría. En el HSM ese instante ocurre dentro de hardware sellado.

**DUKPT** (Derived Unique Key Per Transaction) es el esquema estándar para terminales POS: cada transacción usa una clave derivada distinta, así que comprometer una no compromete las anteriores ni las siguientes.

### Desde Java

```java
// PKCS#11: el estándar para hablar con HSMs
var config = """
    name = payShield
    library = /opt/hsm/lib/libcryptoki.so
    slot = 0
    """;

Provider provider = Security.getProvider("SunPKCS11").configure(config);
Security.addProvider(provider);

KeyStore ks = KeyStore.getInstance("PKCS11", provider);
ks.load(null, pinDelSlot);

// La clave es una referencia: el material nunca llega a la JVM
Key clave = ks.getKey("clave-cifrado-pan", null);

Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding", provider);
cipher.init(Cipher.ENCRYPT_MODE, clave);
byte[] cifrado = cipher.doFinal(pan.getBytes(UTF_8));
```

> [!tip] Alternativas gestionadas
> Si no hay HSM físico: Azure Dedicated HSM / Managed HSM, AWS CloudHSM y AWS KMS ofrecen respaldo por hardware certificado FIPS 140. Para muchos casos alcanzan y evitan operar el aparato.

## 🔒 Cifrado en la aplicación

Cuando sí tenés que guardar el PAN cifrado:

```java
@Converter
public class PanCifradoConverter implements AttributeConverter<String, byte[]> {

    private final Cipher cipher;   // respaldado por HSM o KMS

    @Override
    public byte[] convertToDatabaseColumn(String pan) {
        return cripto.cifrar(pan);          // AES-256-GCM
    }

    @Override
    public String convertToEntityAttribute(byte[] cifrado) {
        return cripto.descifrar(cifrado);
    }
}
```

```java
@Entity
public class TarjetaEntity {

    @Convert(converter = PanCifradoConverter.class)
    @Column(name = "pan_cifrado")
    private String pan;             // cifrado en reposo, transparente en el código

    @Column(name = "pan_bin")       private String bin;        // primeros 6, en claro
    @Column(name = "pan_ultimos4")  private String ultimos4;   // últimos 4, en claro
    @Column(name = "pan_hash")      private String hash;       // HMAC, para buscar
}
```

> [!tip] Para buscar por PAN sin descifrar, usá HMAC — no un hash simple
> El espacio de tarjetas es chico y estructurado: un `SHA-256` de un PAN se rompe por fuerza bruta en minutos. Con **HMAC-SHA256 y una clave secreta** (guardada en el HSM), el atacante que roba la base no puede calcular los hashes.

> [!danger] Usá siempre modo autenticado
> `AES/GCM/NoPadding` o `AES/CBC` **con HMAC**. `AES/ECB` filtra patrones y `AES/CBC` sin autenticación es vulnerable a manipulación del texto cifrado. Y el IV debe ser aleatorio y distinto en cada operación.

## 📋 Los 12 requisitos, resumidos

| # | Requisito | Qué implica en el código |
|---|---|---|
| 1-2 | Firewall y sin contraseñas por defecto | infraestructura |
| **3** | **Proteger los datos almacenados** | cifrado, enmascarado, no guardar SAD |
| **4** | **Cifrar en tránsito** | TLS 1.2+ obligatorio |
| 5-6 | Antimalware y **desarrollo seguro** | SAST, revisión, gestión de CVEs |
| **7-8** | **Acceso mínimo e identificación** | RBAC, MFA, sin cuentas compartidas |
| 9 | Acceso físico | centro de datos |
| **10** | **Registrar y monitorear** | auditoría de todo acceso a datos de tarjeta |
| 11 | Pruebas de seguridad | pentesting, escaneo |
| 12 | Política de seguridad | organizativo |

Los que tocan directamente a un backend son **3, 4, 6, 7, 8 y 10**.

## 🧪 Tests

```java
@Test
void ningun_log_contiene_un_pan_completo() {
    servicio.procesarPago(pagoConTarjeta());
    String logs = capturador.contenido();
    assertFalse(Pattern.compile("\\b\\d{13,19}\\b").matcher(logs).find());
}

@Test
void el_cvv_no_se_persiste_en_ninguna_tabla() {
    servicio.autorizar(solicitudCon("123"));
    assertFalse(jdbc.existeValorEnAlgunaColumna("123"));
}

@Test
void el_pan_se_guarda_cifrado() {
    tarjetaRepo.save(tarjeta("4111111111111111"));
    byte[] crudo = jdbc.leerColumnaCruda("pan_cifrado");
    assertFalse(new String(crudo).contains("4111"));
}

@Test
void la_respuesta_de_la_api_solo_expone_bin_y_ultimos4() { ... }

@Test
void dos_cifrados_del_mismo_pan_dan_resultados_distintos() {
    assertNotEquals(cripto.cifrar(pan), cripto.cifrar(pan));   // IV aleatorio
}
```

## ⚠️ Errores Comunes

> [!warning] Los que generan incidentes reportables
> - **Guardar el CVV** "por si el cliente vuelve a comprar". Prohibido, siempre.
> - **Loguear el request completo** de un pago: el PAN y el CVV terminan en el log.
> - **`SHA-256` simple del PAN**: se rompe por fuerza bruta. Va HMAC con clave.
> - **`AES/ECB`** o CBC sin autenticación.
> - **IV fijo o reutilizado.**
> - **Clave de cifrado en el `application.yml`** o en una variable de entorno.
> - **PAN completo en la respuesta de la API** "porque el front lo necesita". No lo necesita.
> - **Datos reales de tarjeta en ambientes de prueba**: el alcance se extiende a QA.

## 📌 Notas Rápidas
- La forma más barata de cumplir PCI-DSS es **no tocar el dato**. Cada decisión de arquitectura que saca el PAN de tu sistema ahorra meses de certificación.
- El nivel de exigencia depende del volumen anual de transacciones (niveles 1 a 4) y el cuestionario aplicable (SAQ) depende de cómo manejás los datos. Un comercio con iframe llena un SAQ mínimo; uno que recibe el PAN llena el completo.
- PCI-DSS v4.0 es la versión vigente; endureció requisitos de MFA, gestión de scripts en páginas de pago y monitoreo.
- En una entrevista, saber que **SAD no se almacena nunca ni cifrado** y que el enmascarado es primeros 6 / últimos 4 demuestra que conocés el estándar de verdad.

## 🔗 Relaciones
- **Módulo**: [[Banca y Fintech]]
- **Anterior**: [[10 - Cierre de Dia y Batch]]
- **Siguiente**: [[12 - Auditoria y Trazabilidad]]
- **Se conecta con**: [[07 - Medios de Pago y ISO 8583]], [[08 - Criptografia en Java]], [[07 - OWASP Top 10 en Java]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

