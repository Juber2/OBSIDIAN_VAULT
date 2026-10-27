---
tags:
  - Java
  - Senior
  - Seguridad
Creado: 2026-09-08
Relacionado:
  - Senior Java Engineer
  - 11 - PCI-DSS y HSM
Palabras clave:
  - JCA, AES-GCM, BCrypt, Argon2, HMAC, SecureRandom, KeyStore, firma digital
---

## 🧠 Concepto

La **JCA** (Java Cryptography Architecture) es la API estándar de criptografía de Java. Es potente, y también es fácil de usar mal: casi todas sus opciones por defecto son las inseguras, por compatibilidad histórica.

> [!danger] La regla número uno
> **No inventes criptografía.** No diseñes tu propio esquema de cifrado, no combines primitivas por intuición, no "mejores" un algoritmo. Usá construcciones estándar con parámetros recomendados.
> La segunda regla: si no sabés qué modo de operación usar, es `AES/GCM/NoPadding`.

## 🧩 Las cuatro herramientas

| Necesito | Herramienta | Reversible |
|---|---|---|
| Verificar integridad | hash (`MessageDigest`) | ❌ |
| Verificar integridad y autenticidad | **HMAC** (`Mac`) | ❌ |
| Ocultar datos y poder recuperarlos | **cifrado** (`Cipher`) | ✅ con la clave |
| Guardar contraseñas | **KDF** (BCrypt, Argon2) | ❌ a propósito |

> [!warning] Un hash no es cifrado y una contraseña no se cifra
> Son los dos errores conceptuales más frecuentes. El hash es unidireccional y sin clave; el cifrado es reversible con clave. Y una contraseña **nunca se cifra**: se deriva con una función lenta y con sal.

## 🔑 Contraseñas

```java
// ❌ Hash rápido: una GPU prueba miles de millones por segundo
String hash = DigestUtils.sha256Hex(password);
String hash = DigestUtils.sha256Hex(password + sal);    // tampoco alcanza

// ✅ Función diseñada para ser LENTA
var encoder = new BCryptPasswordEncoder(12);
String hash = encoder.encode(password);
boolean ok = encoder.matches(passwordIngresado, hash);
```

| Función | Resistente a GPU | Resistente a memoria | Recomendación |
|---|---|---|---|
| `MD5`, `SHA-*` | ❌ | ❌ | **nunca para contraseñas** |
| PBKDF2 | parcial | ❌ | aceptable si lo exige una norma |
| **BCrypt** | ✅ | parcial | **buen default** |
| **Argon2id** | ✅ | ✅ | **el más recomendado hoy** |
| scrypt | ✅ | ✅ | buena alternativa |

```java
// Argon2 con Spring Security
var encoder = new Argon2PasswordEncoder(16, 32, 1, 1 << 14, 2);
//                                       sal, hash, paralelismo, memoria(KB), iteraciones
```

> [!tip] El coste se calibra por tiempo, no por número
> Se ajusta el factor de trabajo hasta que verificar una contraseña tarde **entre 200 y 500 ms** en tu hardware de producción. Eso hace la fuerza bruta inviable sin degradar el login.
> El coste 12 de BCrypt suele caer en ese rango; conviene medirlo, no asumirlo.

```java
// Migración transparente al cambiar de algoritmo
var encoder = new DelegatingPasswordEncoder("argon2", Map.of(
    "argon2", new Argon2PasswordEncoder(...),
    "bcrypt", new BCryptPasswordEncoder(12)));
// Los hashes viejos ({bcrypt}$2a$...) se siguen verificando;
// los nuevos se guardan con argon2.
```

## 🔒 Cifrado simétrico

```java
public class CifradoAes {

    private static final int TAM_IV = 12;         // 96 bits: recomendado para GCM
    private static final int TAM_TAG = 128;       // bits de autenticación

    public byte[] cifrar(byte[] datos, SecretKey clave) throws Exception {
        byte[] iv = new byte[TAM_IV];
        SecureRandom.getInstanceStrong().nextBytes(iv);     // ← IV ALEATORIO por operación

        var cipher = Cipher.getInstance("AES/GCM/NoPadding");
        cipher.init(Cipher.ENCRYPT_MODE, clave, new GCMParameterSpec(TAM_TAG, iv));

        byte[] cifrado = cipher.doFinal(datos);

        // El IV no es secreto: viaja junto al texto cifrado
        return ByteBuffer.allocate(iv.length + cifrado.length)
            .put(iv).put(cifrado).array();
    }

    public byte[] descifrar(byte[] entrada, SecretKey clave) throws Exception {
        var buffer = ByteBuffer.wrap(entrada);
        byte[] iv = new byte[TAM_IV];
        buffer.get(iv);
        byte[] cifrado = new byte[buffer.remaining()];
        buffer.get(cifrado);

        var cipher = Cipher.getInstance("AES/GCM/NoPadding");
        cipher.init(Cipher.DECRYPT_MODE, clave, new GCMParameterSpec(TAM_TAG, iv));

        return cipher.doFinal(cifrado);      // lanza AEADBadTagException si fue manipulado
    }
}
```

### Los modos de operación

| Modo | Confidencialidad | Autenticidad | Veredicto |
|---|---|---|---|
| **ECB** | ❌ filtra patrones | ❌ | **nunca** |
| CBC | ✅ | ❌ | solo con HMAC aparte |
| CTR | ✅ | ❌ | ídem |
| **GCM** | ✅ | ✅ | ✅ **el default correcto** |
| ChaCha20-Poly1305 | ✅ | ✅ | ✅ alternativa |

> [!danger] `AES/ECB` filtra la estructura del texto plano
> Cifra cada bloque de forma independiente, así que bloques iguales dan cifrados iguales. La demostración clásica es cifrar una imagen con ECB: la silueta sigue siendo perfectamente visible.
> Y peor: `Cipher.getInstance("AES")` **usa ECB por defecto**. Siempre hay que especificar el modo completo.

> [!danger] El IV nunca se reutiliza con la misma clave en GCM
> Reutilizar el par (clave, IV) en GCM no solo filtra información: **permite recuperar la clave de autenticación** y falsificar mensajes. Es el fallo más catastrófico posible con este modo.
> El IV se genera aleatoriamente en cada operación y viaja junto al texto cifrado. No es secreto; solo tiene que ser único.

## ✍️ HMAC

```java
public String hmac(String datos, SecretKey clave) throws Exception {
    var mac = Mac.getInstance("HmacSHA256");
    mac.init(clave);
    return HexFormat.of().formatHex(mac.doFinal(datos.getBytes(UTF_8)));
}
```

Casos de uso típicos: firmar webhooks, generar índices de búsqueda sobre datos cifrados, verificar tokens.

```java
// Buscar por PAN sin descifrar: HMAC, no hash simple
String indice = hmac(pan, claveDeIndice);      // guardado junto al PAN cifrado
```

> [!danger] Para datos de bajo espacio de búsqueda, hash simple no protege
> Un `SHA-256` de un número de tarjeta se rompe por fuerza bruta en minutos: el espacio de PAN válidos es chico y estructurado. Con **HMAC y una clave secreta**, quien roba la base no puede calcular los hashes sin la clave.
> Lo mismo aplica a documentos de identidad, teléfonos y correos — ver [[11 - PCI-DSS y HSM]].

```java
// Comparar en tiempo constante: evita ataques de temporización
MessageDigest.isEqual(hmacEsperado, hmacRecibido);    // ✅
Arrays.equals(hmacEsperado, hmacRecibido);            // ⚠️ sale al primer byte distinto
esperado.equals(recibido);                            // ❌ igual problema
```

## 🎲 Aleatoriedad

```java
// ❌ Predecible: no es criptográfico
new Random().nextInt();
Math.random();
UUID.randomUUID();      // ⚠️ usa SecureRandom, pero verificá si es tu caso

// ✅
SecureRandom aleatorio = SecureRandom.getInstanceStrong();
byte[] token = new byte[32];
aleatorio.nextBytes(token);
String valor = Base64.getUrlEncoder().withoutPadding().encodeToString(token);
```

> [!warning] `getInstanceStrong()` puede bloquear en Linux
> Usa una fuente de entropía bloqueante. En un contenedor recién arrancado, con poca entropía acumulada, la primera llamada puede tardar segundos.
> Alternativas: `new SecureRandom()` (no bloqueante y criptográficamente seguro en la práctica), o configurar `-Djava.security.egd=file:/dev/urandom`.

> [!danger] `SecureRandom` sembrado antes de una instantánea da la misma secuencia
> Es el riesgo específico de AWS Lambda SnapStart y CRaC: todas las instancias restauradas comparten el estado del generador. Tokens predecibles en todas ellas.
> Hay que resembrar en `afterRestore` — ver [[11 - Serverless con Java]].

## 🗝️ Gestión de claves

```java
// ❌ La clave en el código o en la configuración
private static final String CLAVE = "miClaveSecreta123";

// ✅ Desde el almacén de secretos, con identidad gestionada
var clave = new SecretKeySpec(
    Base64.getDecoder().decode(secretos.obtener("clave-cifrado-pan")), "AES");

// ✅✅ Mejor: la clave nunca sale del HSM o del KMS
Provider provider = Security.getProvider("SunPKCS11").configure(configHsm);
KeyStore ks = KeyStore.getInstance("PKCS11", provider);
Key clave = ks.getKey("clave-pan", null);      // una referencia, no el material
```

### Rotación

```java
// La clave usada se identifica en el dato cifrado
public record DatoCifrado(String versionClave, byte[] contenido) { }

public byte[] descifrar(DatoCifrado dato) {
    SecretKey clave = claves.porVersion(dato.versionClave());    // la vieja sigue disponible
    return cifrado.descifrar(dato.contenido(), clave);
}
```

> [!tip] Versionar la clave hace posible rotar sin recifrar todo
> Los datos nuevos se cifran con la clave actual; los viejos se siguen descifrando con la suya. El recifrado masivo, si hace falta, se hace en segundo plano y sin ventana de indisponibilidad.
> Sin versión de clave, rotar significa recifrar toda la base de una vez.

## ✒️ Firma digital

```java
// Firmar
var firma = Signature.getInstance("SHA256withRSA");
firma.initSign(clavePrivada);
firma.update(datos);
byte[] resultado = firma.sign();

// Verificar
var verificador = Signature.getInstance("SHA256withRSA");
verificador.initVerify(clavePublica);
verificador.update(datos);
boolean valida = verificador.verify(resultado);
```

| | HMAC | Firma digital |
|---|---|---|
| Clave | compartida | par público/privado |
| Quién puede verificar | quien tiene la clave | cualquiera con la pública |
| No repudio | ❌ | ✅ |
| Velocidad | rápida | más lenta |

> [!tip] Para webhooks salientes, firma digital; para internos, HMAC
> Si el receptor es un tercero, una firma con clave pública le permite verificar sin que vos le compartas un secreto — y te da no repudio. Entre servicios propios, HMAC alcanza y es más rápido.

## 📋 Elegir el algoritmo

| Necesito | Usar | Evitar |
|---|---|---|
| Contraseñas | Argon2id, BCrypt | MD5, SHA-*, PBKDF2 débil |
| Cifrado simétrico | AES-256-GCM | ECB, CBC sin HMAC, DES, 3DES |
| Cifrado asimétrico | RSA-2048+, ECC P-256+ | RSA-1024 |
| Hash de integridad | SHA-256, SHA-3 | MD5, SHA-1 |
| Autenticación de mensaje | HMAC-SHA256 | hash concatenado con la clave |
| Firma | ECDSA P-256, RSA-PSS | RSA PKCS#1 v1.5 en diseños nuevos |
| Aleatoriedad | `SecureRandom` | `Random`, `Math.random()` |

## 🧪 Tests

```java
@Test
void dos_cifrados_del_mismo_dato_dan_resultados_distintos() {
    assertNotEquals(
        Arrays.toString(cifrado.cifrar(datos, clave)),
        Arrays.toString(cifrado.cifrar(datos, clave)));    // IV aleatorio ✅
}

@Test
void manipular_el_texto_cifrado_es_detectado() {
    byte[] c = cifrado.cifrar(datos, clave);
    c[c.length - 5] ^= 0x01;                                // un bit cambiado
    assertThrows(AEADBadTagException.class, () -> cifrado.descifrar(c, clave));
}

@Test
void el_hash_de_contrasena_es_lento_a_proposito() {
    long inicio = System.nanoTime();
    encoder.encode("contrasena");
    var duracion = Duration.ofNanos(System.nanoTime() - inicio);
    assertTrue(duracion.toMillis() > 100);      // si es rápido, el coste es muy bajo
}

@Test
void la_misma_contrasena_genera_hashes_distintos() {
    assertNotEquals(encoder.encode("x"), encoder.encode("x"));    // sal aleatoria ✅
}

@Test
void un_dato_cifrado_con_la_clave_v1_se_descifra_tras_rotar() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que producen vulnerabilidades reales
> - **Hash simple para contraseñas**: se rompen con GPU en minutos.
> - **`Cipher.getInstance("AES")`**: usa ECB por defecto.
> - **CBC sin autenticación**: el cifrado se puede manipular.
> - **IV fijo o reutilizado** en GCM: catastrófico.
> - **`Random`** en vez de `SecureRandom` para tokens.
> - **Clave en el código o en la configuración.**
> - **Sin versión de clave**: rotar exige recifrar todo.
> - **`equals` para comparar HMAC**: ataque de temporización.
> - **Inventar el esquema criptográfico** en vez de usar uno estándar.
> - **`SecureRandom` sembrado antes de una instantánea** (SnapStart/CRaC).

## 📌 Notas Rápidas
- Los tres defaults que hay que memorizar: **Argon2id o BCrypt** para contraseñas, **AES-256-GCM** para cifrar, **HMAC-SHA256** para autenticar. Con eso se cubre el 95% de los casos de un backend.
- Que `Cipher.getInstance("AES")` use ECB es una trampa histórica de la API: la opción por defecto es la insegura. Siempre hay que especificar modo y relleno completos.
- Versionar las claves desde el principio cuesta un campo y evita un proyecto entero de recifrado cuando toque rotar.
- Se aplica en [[Fase 3 - Seguridad y Auditoria]] y se profundiza en [[11 - PCI-DSS y HSM]].

## 🔗 Relaciones
- **Módulo**: [[Senior Java Engineer]]
- **Anterior**: [[07 - OWASP Top 10 en Java]]
- **Siguiente**: [[09 - ADR y Documentacion]]
- **Se conecta con**: [[11 - PCI-DSS y HSM]], [[04 - Configuracion y Secretos]], [[02 - JWT]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

