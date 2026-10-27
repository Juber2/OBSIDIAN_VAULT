---
tags:
  - Java
  - Senior
  - Seguridad
Creado: 2026-09-08
Relacionado:
  - Senior Java Engineer
  - 03 - Seguridad Backend Java
Palabras clave:
  - OWASP, IDOR, inyeccion, SSRF, deserializacion, XXE, dependencias, configuracion
---

## 🧠 Concepto

El **OWASP Top 10** es la lista de las categorías de vulnerabilidad más críticas en aplicaciones web, publicada por la Open Worldwide Application Security Project. Esta nota la recorre desde el código Java: qué se ve mal, cómo se arregla y cómo se detecta.

## A01 · Control de acceso roto

**La número uno, y por lejos la más común en backends.**

```java
// ❌ IDOR: valida el rol, no el dueño del recurso
@GetMapping("/api/v1/cuentas/{id}")
@PreAuthorize("hasRole('CLIENTE')")
public CuentaResponse obtener(@PathVariable UUID id) {
    return service.obtener(id);      // ¡cualquier cliente ve cualquier cuenta!
}
```

```java
// ✅ Autorización por recurso
@GetMapping("/api/v1/cuentas/{id}")
@PreAuthorize("@accesoCuenta.puedeAcceder(#id, authentication)")
public CuentaResponse obtener(@PathVariable UUID id) { ... }
```

```java
@Component("accesoCuenta")
public class EvaluadorAccesoCuenta {
    public boolean puedeAcceder(UUID cuentaId, Authentication auth) {
        var principal = (UsuarioAutenticado) auth.getPrincipal();
        if (principal.tieneAlgunRol("OPERADOR", "AUDITOR")) return true;
        return cuentas.findById(cuentaId)
            .map(c -> c.getClienteId().equals(principal.clienteId()))
            .orElse(false);
    }
}
```

> [!danger] Las tres preguntas que hay que separar
> **¿Quién sos?** (autenticación) · **¿Qué podés hacer?** (rol) · **¿Sobre qué?** (recurso).
> La tercera es la que se olvida. Cambiar el UUID en la URL y ver los datos de otro cliente es el fallo más frecuente y más fácil de explotar que existe.

Otras variantes de la misma categoría:

```java
// ❌ Confiar en un id que viene del cliente
public void transferir(@RequestBody TransferenciaRequest req) {
    servicio.transferir(req.cuentaOrigen(), ...);    // ¿es del usuario autenticado?
}

// ❌ Endpoint interno expuesto sin protección
@GetMapping("/internal/usuarios")   // accesible desde internet

// ❌ Escalada por asignación masiva
@PostMapping("/usuarios")
public void crear(@RequestBody UsuarioEntity u) { repo.save(u); }
// el cliente manda {"rol": "ADMIN"} y se lo asigna
```

## A02 · Fallos criptográficos

```java
// ❌ Contraseñas con hash simple: se rompen en minutos
String hash = DigestUtils.sha256Hex(password);

// ✅ Función diseñada para ser lenta
var encoder = new BCryptPasswordEncoder(12);
String hash = encoder.encode(password);
```

```java
// ❌ AES en modo ECB: filtra patrones del texto plano
Cipher.getInstance("AES/ECB/PKCS5Padding");

// ❌ CBC sin autenticación: el texto cifrado se puede manipular
Cipher.getInstance("AES/CBC/PKCS5Padding");

// ✅ Modo autenticado
Cipher.getInstance("AES/GCM/NoPadding");
```

Detalle en [[08 - Criptografia en Java]].

> [!warning] Datos sensibles en tránsito y en reposo
> TLS 1.2 mínimo (preferible 1.3), incluso entre servicios internos: "es la red privada" no es un control en un modelo de confianza cero. Y nada de datos sensibles en URLs — quedan en logs de proxy, historial y cabeceras `Referer`.

## A03 · Inyección

```java
// ❌ SQL por concatenación
String sql = "SELECT * FROM cuenta WHERE numero = '" + numero + "'";
jdbc.query(sql, ...);
// numero = "' OR '1'='1" → devuelve todas las cuentas
```

```java
// ✅ Parámetros
jdbc.query("SELECT * FROM cuenta WHERE numero = ?", rowMapper, numero);

// ✅ En JPA
@Query("SELECT c FROM Cuenta c WHERE c.numero = :numero")
Optional<Cuenta> buscar(@Param("numero") String numero);
```

> [!danger] JPQL también se puede inyectar
> ```java
> em.createQuery("SELECT c FROM Cuenta c WHERE c.numero = '" + numero + "'");  // ❌
> ```
> Usar JPA no protege por sí solo: protege usar **parámetros**. Y hay un caso que los parámetros no cubren:
> ```java
> // ❌ El nombre de columna no se puede parametrizar
> String sql = "SELECT * FROM cuenta ORDER BY " + columna;
> ```
> Ahí hay que validar contra una **lista blanca**:
> ```java
> private static final Set<String> COLUMNAS = Set.of("numero", "creado_en", "saldo");
> if (!COLUMNAS.contains(columna)) throw new ParametroInvalidoException(columna);
> ```

Otras inyecciones que aplican en Java:

```java
// Inyección de comandos
Runtime.getRuntime().exec("convert " + archivoDelUsuario);     // ❌

// Recorrido de rutas
Files.readAllBytes(Path.of("/datos/", nombreDelUsuario));      // ❌ "../../etc/passwd"
// ✅
Path base = Path.of("/datos").toRealPath();
Path destino = base.resolve(nombre).normalize();
if (!destino.startsWith(base)) throw new RutaInvalidaException();

// Inyección en logs
log.info("Login de " + usuario);      // ❌ el usuario mete saltos de línea y falsea entradas
log.atInfo().addKeyValue("usuario", usuario).log("login");   // ✅ estructurado
```

## A04 · Diseño inseguro

No es un bug de implementación: es una funcionalidad pensada sin considerar el abuso.

| Ejemplo | Falta |
|---|---|
| Recuperar contraseña con pregunta secreta | el dato es público en redes sociales |
| Sin límite de intentos de login | fuerza bruta libre |
| Transferencias sin segundo factor | el robo de sesión basta para vaciar la cuenta |
| Enumerar usuarios por el mensaje de error | "usuario no existe" vs "contraseña incorrecta" |
| Sin límite de tasa en operaciones caras | denegación de servicio trivial |

> [!tip] La defensa es modelar amenazas al diseñar, no auditar al final
> Preguntarse *"¿cómo abusaría alguien de esta funcionalidad?"* en la etapa de diseño cuesta media hora. Descubrirlo en una auditoría cuesta un rediseño.

## A05 · Configuración incorrecta

```yaml
# ❌ Todo Actuator expuesto
management:
  endpoints:
    web:
      exposure:
        include: "*"        # /env, /heapdump, /configprops accesibles
```

```yaml
# ✅ Solo lo necesario
management:
  endpoints:
    web:
      exposure:
        include: health, prometheus
  endpoint:
    health:
      show-details: never
```

```java
// ❌ Devolver el stacktrace al cliente
@ExceptionHandler(Exception.class)
public ResponseEntity<String> error(Exception ex) {
    return ResponseEntity.status(500).body(ex.getMessage() + Arrays.toString(ex.getStackTrace()));
}
```

| Configuración a revisar | Correcto |
|---|---|
| Credenciales por defecto | cambiadas |
| Perfiles de desarrollo en producción | desactivados |
| CORS con `*` | orígenes explícitos |
| Directorios listables | desactivado |
| Cabeceras de seguridad | HSTS, X-Content-Type-Options, CSP |
| Mensajes de error | genéricos hacia afuera |

```java
http.headers(h -> h
    .httpStrictTransportSecurity(hsts -> hsts.maxAgeInSeconds(31536000))
    .contentTypeOptions(Customizer.withDefaults())
    .frameOptions(FrameOptionsConfig::deny));
```

## A06 · Componentes vulnerables

```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>
    </configuration>
</plugin>
```

```yaml
- name: Analizar dependencias
  run: ./mvnw org.owasp:dependency-check-maven:check
```

> [!danger] Log4Shell (CVE-2021-44228) es el ejemplo canónico
> Una biblioteca de logging permitía ejecución remota de código a través de un mensaje logueado. Miles de aplicaciones vulnerables sin haber escrito una línea insegura.
> **La única defensa es saber qué dependencias tenés y actualizarlas.** Un SBOM (inventario de componentes) generado en cada build permite responder "¿estoy afectado?" en minutos en vez de días.

## A07 · Fallos de autenticación

```java
// ❌ Revela si el usuario existe
if (usuario == null) throw new UsuarioNoEncontradoException();
if (!coincide(password)) throw new PasswordIncorrectoException();

// ✅ Mismo mensaje y tiempo de respuesta similar
if (usuario == null || !encoder.matches(password, usuario.hash())) {
    encoder.matches(password, HASH_FALSO);      // iguala el tiempo
    throw new CredencialesInvalidasException();  // mensaje único
}
```

| Control | Implementación |
|---|---|
| Bloqueo por intentos | contador + ventana temporal |
| Segundo factor | obligatorio en operaciones sensibles |
| Sesiones cortas | access token ≤ 15 min |
| Rotación de refresh token | con detección de reuso |
| Invalidar al cambiar contraseña | revocar todas las sesiones |
| Contraseñas filtradas | contrastar contra listas conocidas |

> [!tip] La rotación de refresh tokens con detección de reuso
> Si un token ya rotado se vuelve a usar, significa que alguien lo robó: se revoca **toda la cadena** del usuario. Está en la guía de buenas prácticas de OAuth 2.0 y se implementa en [[Fase 3 - Seguridad y Auditoria]].

## A08 · Fallos de integridad

```java
// ❌ Deserialización de datos no confiables: ejecución remota de código
var ois = new ObjectInputStream(request.getInputStream());
Object obj = ois.readObject();
```

> [!danger] La deserialización nativa de Java es peligrosa por diseño
> Deserializar bytes de un origen no confiable puede ejecutar código arbitrario durante el proceso, antes de que veas el objeto. No hay forma segura de hacerlo con datos de un tercero.
> **La regla: no deserialices formato binario de Java desde fuentes externas. Usá JSON con tipos concretos.**

```java
// ⚠️ Jackson con tipado polimórfico habilitado: mismo riesgo
mapper.enableDefaultTyping();                    // ❌ nunca
mapper.activateDefaultTyping(validador, ...);    // solo con lista blanca estricta
```

```xml
<!-- XXE: entidades externas en XML -->
```

```java
var factory = DocumentBuilderFactory.newInstance();
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
factory.setXIncludeAware(false);
factory.setExpandEntityReferences(false);
```

> [!warning] XXE es muy relevante si procesás ISO 20022
> Esos mensajes son XML de terceros. Un parser sin configurar puede leer archivos del servidor o hacer peticiones internas — ver [[08 - ISO 20022 y SWIFT]].

## A09 · Fallos de registro y monitoreo

```java
// ❌ Se traga el error: el ataque es invisible
try { autenticar(usuario, password); } catch (Exception e) { }
```

| Evento | ¿Se registra? |
|---|---|
| Login fallido | ✅ |
| Acceso denegado a un recurso ajeno | ✅ **crítico** |
| Cambio de contraseña o de rol | ✅ |
| Operaciones con dinero | ✅ |
| Exportación masiva de datos | ✅ |

> [!tip] Los rechazos son la señal, no los éxitos
> Diez accesos denegados seguidos sobre cuentas ajenas es el patrón más claro de un ataque en curso. Si solo registrás lo que salió bien, ese patrón es invisible — ver [[12 - Auditoria y Trazabilidad]].

```java
// ❌ Loguear el request completo del login
log.debug("Request: {}", request);      // la contraseña queda en texto plano
```

## A10 · SSRF

El servidor hace una petición a una URL que controla el usuario.

```java
// ❌ El atacante pone una URL interna
@PostMapping("/webhooks")
public void configurar(@RequestBody String url) {
    restTemplate.postForEntity(url, payload, Void.class);
}
// url = "http://169.254.169.254/latest/meta-data/iam/security-credentials/"
// → devuelve las credenciales de la instancia en la nube 💥
```

```java
// ✅ Lista blanca y validación de destino
public void configurarWebhook(String url) {
    URI uri = URI.create(url);

    if (!"https".equals(uri.getScheme()))    throw new UrlInvalidaException("solo https");
    if (!DOMINIOS_PERMITIDOS.contains(uri.getHost())) throw new UrlInvalidaException();

    InetAddress ip = InetAddress.getByName(uri.getHost());
    if (ip.isLoopbackAddress() || ip.isSiteLocalAddress() || ip.isLinkLocalAddress()) {
        throw new UrlInvalidaException("destino interno no permitido");
    }
    // + resolver DNS una sola vez y usar la IP, para evitar re-binding
}
```

> [!danger] SSRF en la nube es especialmente grave
> El servicio de metadatos de la instancia (`169.254.169.254`) entrega credenciales temporales. Un SSRF sin filtrar convierte una funcionalidad inocente en un robo de credenciales de infraestructura.
> Es una razón más para usar IMDSv2 y endpoints privados — ver [[05 - Identidad y Acceso]].

## 🛠️ Herramientas

```yaml
- name: SAST
  run: ./mvnw com.github.spotbugs:spotbugs-maven-plugin:check    # con find-sec-bugs

- name: Dependencias
  run: ./mvnw org.owasp:dependency-check-maven:check

- name: Secretos
  uses: gitleaks/gitleaks-action@v2

- name: Imagen
  uses: aquasecurity/trivy-action@master
```

| Herramienta | Qué encuentra |
|---|---|
| **find-sec-bugs** | patrones inseguros en el código Java |
| **OWASP Dependency-Check** | CVE en dependencias |
| **gitleaks** | secretos en el repositorio |
| **Trivy** | vulnerabilidades en la imagen |
| **OWASP ZAP** | análisis dinámico de la API |

## 🧪 Tests de seguridad

```java
@Test
void un_cliente_no_puede_ver_la_cuenta_de_otro() {
    var token = autenticarComo(clienteA);
    assertEquals(403, api.conToken(token).get("/api/v1/cuentas/" + cuentaDeB).statusCode());
}

@Test
void la_inyeccion_sql_no_devuelve_datos() {
    var r = api.get("/api/v1/cuentas?numero=' OR '1'='1");
    assertTrue(r.jsonPath().getList("datos").isEmpty());
}

@Test
void el_mensaje_de_login_no_revela_si_el_usuario_existe() {
    var m1 = api.post("/login", Map.of("usuario", "inexistente", "password", "x")).body();
    var m2 = api.post("/login", Map.of("usuario", "real", "password", "mal")).body();
    assertEquals(m1, m2);
}

@Test
void los_endpoints_sensibles_de_actuator_no_estan_expuestos() {
    assertEquals(404, api.get("/actuator/env").statusCode());
    assertEquals(404, api.get("/actuator/heapdump").statusCode());
}

@Test
void ningun_log_contiene_datos_sensibles() { ... }

@Test
void una_url_interna_es_rechazada_en_el_webhook() {
    assertThrows(UrlInvalidaException.class,
        () -> servicio.configurarWebhook("http://169.254.169.254/"));
}
```

## ✅ Lista de revisión

- [ ] Toda consulta valida el **dueño** del recurso, no solo el rol.
- [ ] Cero concatenación en SQL, JPQL, comandos o rutas.
- [ ] Contraseñas con BCrypt o Argon2, nunca con hash simple.
- [ ] Cifrado con modo autenticado (GCM), IV aleatorio por operación.
- [ ] Actuator acotado a `health` y métricas.
- [ ] Sin stacktraces hacia el cliente.
- [ ] Análisis de dependencias en el pipeline, con umbral que falla el build.
- [ ] Mensaje de login único, sin enumeración de usuarios.
- [ ] Bloqueo por intentos fallidos.
- [ ] Sin deserialización binaria de datos externos.
- [ ] Parsers XML con XXE desactivado.
- [ ] Se auditan los **rechazos**, no solo los éxitos.
- [ ] Toda URL provista por el usuario, validada contra lista blanca y rangos internos.

## 📌 Notas Rápidas
- Si tuvieras que revisar una sola categoría en un backend, sería **A01**: el control de acceso roto es el más común, el más fácil de explotar y el que más datos expone.
- El error de IDOR aparece porque la autenticación y la autorización por rol "se ven" hechas: hay `@PreAuthorize`, hay token, hay roles. Lo que falta es invisible en la revisión de código.
- El Top 10 es una lista de **categorías**, no de bugs concretos. Sirve como marco de revisión, no como checklist exhaustivo.
- Se implementa en [[Fase 3 - Seguridad y Auditoria]] y se relaciona con [[11 - PCI-DSS y HSM]].

## 🔗 Relaciones
- **Módulo**: [[Senior Java Engineer]]
- **Base**: [[03 - Seguridad Backend Java]], [[Spring Security]]
- **Siguiente**: [[08 - Criptografia en Java]]
- **Se conecta con**: [[05 - Identidad y Acceso]], [[12 - Auditoria y Trazabilidad]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

