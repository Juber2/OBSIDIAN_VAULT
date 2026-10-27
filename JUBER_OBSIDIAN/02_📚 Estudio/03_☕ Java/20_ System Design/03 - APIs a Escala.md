---
tags:
  - Java
  - System Design
  - APIs
Creado: 2026-09-08
Relacionado:
  - System Design
  - 04 - Diseno de APIs Senior
Palabras clave:
  - REST, gRPC, GraphQL, paginacion por cursor, versionado, lote, compresion, contrato
---

## 🧠 Concepto

Una API que funciona con mil usuarios puede volverse el cuello de botella con un millón. Las decisiones que lo determinan —paginación, versionado, granularidad, formato— se toman al diseñar el contrato y son **caras de cambiar después**, porque hay clientes que no controlás.

## 🔀 Elegir el estilo

| | REST | gRPC | GraphQL |
|---|---|---|---|
| Formato | JSON sobre HTTP | Protobuf sobre HTTP/2 | JSON con consulta |
| Rendimiento | bueno | **excelente** | bueno |
| Descubrible | ✅ | necesita el `.proto` | ✅ con introspección |
| Desde el navegador | ✅ | necesita proxy | ✅ |
| Contrato tipado | con OpenAPI | ✅ nativo | ✅ nativo |
| Caché HTTP | ✅ | ❌ | ❌ (todo es POST) |
| Flujos bidireccionales | ❌ | ✅ | suscripciones |

```
API pública o para terceros    → REST  (ver [[14 - Open Banking]])
Servicio a servicio interno    → gRPC  (menos latencia y menos bytes)
Cliente que arma sus consultas → GraphQL
Streaming                       → gRPC o WebSockets
```

> [!warning] GraphQL traslada el problema al servidor
> El cliente pide exactamente lo que necesita, y esa flexibilidad puede generar consultas carísimas: una petición anidada puede disparar cientos de consultas a la base (el N+1 clásico de GraphQL).
> Necesita defensas propias: análisis de complejidad, límite de profundidad, y **DataLoader** para agrupar consultas. No es gratis.

## 📄 Paginación

La decisión de escalabilidad más importante de una API de lectura.

```java
// ❌ OFFSET: el costo crece con la página
@Query("SELECT m FROM Movimiento m WHERE m.cuentaId = :id ORDER BY m.id DESC")
Page<Movimiento> buscar(UUID id, Pageable pageable);   // OFFSET 100000 LIMIT 50

// ✅ Cursor: costo constante
@Query("""
    SELECT m FROM Movimiento m
    WHERE m.cuentaId = :id AND m.id < :cursor
    ORDER BY m.id DESC
    """)
List<Movimiento> buscar(UUID id, Long cursor, Limit limit);
```

| | OFFSET | Cursor |
|---|---|---|
| Costo de la página N | crece linealmente | **constante** |
| Saltar a una página arbitraria | ✅ | ❌ |
| Resultados estables al insertar datos | ❌ duplica u omite filas | ✅ |
| Total de páginas | ✅ | ❌ (costoso) |

> [!danger] `OFFSET 100000` lee y descarta 100.000 filas
> La base tiene que recorrerlas para llegar a la que te interesa. En una tabla de movimientos con años de historia, las últimas páginas tardan segundos.
> Y hay un problema peor: **si alguien inserta una fila mientras paginás, los resultados se desplazan** y ves registros repetidos o te salteás otros.

```json
{
  "datos": [ ... ],
  "paginacion": {
    "siguienteCursor": "eyJpZCI6MTIzNDU2fQ==",
    "hayMas": true
  }
}
```

> [!tip] Codificá el cursor en Base64
> No para ocultarlo, sino para que sea **opaco**: el cliente lo trata como una cadena sin significado y no depende de su formato interno. Eso te deja cambiar la estrategia de paginación después sin romper a nadie.

## 🔢 Versionado

| Estrategia | Ejemplo | Comentario |
|---|---|---|
| **En la ruta** | `/api/v1/cuentas` | explícito, fácil de rutear ✅ |
| En cabecera | `Accept: application/vnd.api.v1+json` | limpio, menos visible |
| Por parámetro | `?version=1` | fácil de olvidar |
| Sin versión | — | solo si controlás todos los clientes |

> [!warning] Poné la versión desde el primer día
> Agregarla después obliga a que todos los clientes cambien sus URLs. Es un cambio de una línea al empezar y un proyecto de coordinación seis meses más tarde.

### Qué rompe y qué no

| Cambio | ¿Rompe? |
|---|---|
| Agregar un campo a la respuesta | ❌ (si el cliente ignora lo desconocido) |
| Agregar un parámetro opcional | ❌ |
| Quitar o renombrar un campo | ✅ |
| Cambiar el tipo de un campo | ✅ |
| Hacer obligatorio un parámetro | ✅ |
| Cambiar el código de estado | ✅ |
| Cambiar el orden por defecto | ✅ silenciosamente |

> [!danger] Los cambios que rompen en silencio son los peores
> Cambiar el orden por defecto de una lista no lanza ningún error: el cliente sigue funcionando y muestra los datos mal. Nadie se entera hasta que alguien nota que el extracto está al revés.

```java
// Estrategia de transición: agregar el campo nuevo, mantener el viejo, deprecar
public record CuentaResponse(
        UUID id,
        @Deprecated(since = "v1.4") String nombre,      // se retira en v2
        String titularNombre,                           // el nuevo
        BigDecimal saldo) {}
```

## 📦 Operaciones por lote

```java
// ❌ El cliente hace N llamadas: N × latencia de red
for (UUID id : ids) { api.obtenerCuenta(id); }

// ✅ Una llamada
POST /api/v1/cuentas/consulta-masiva
{ "ids": ["...", "...", "..."] }
```

> [!tip] Un endpoint por lote elimina el N+1 de red
> Es el mismo problema que el N+1 de JPA, con latencias mucho peores: cada llamada remota cuesta cientos de veces más que una consulta local — ver [[01 - Falacias de la Computacion Distribuida]].

```json
// Respuesta parcial: algunos ítems pueden fallar
{
  "resultados": [
    { "id": "a", "estado": "ok", "datos": { ... } },
    { "id": "b", "estado": "no_encontrado" },
    { "id": "c", "estado": "sin_permiso" }
  ]
}
```

> [!warning] Limitá el tamaño del lote
> Sin límite, un cliente puede pedir 100.000 elementos en una llamada y agotar la memoria del servidor. Entre 50 y 500 suele ser razonable, y el límite se documenta y se valida.

## 🗜️ Reducir el tamaño

```yaml
server:
  compression:
    enabled: true
    mime-types: application/json
    min-response-size: 1024
```

```
GET /api/v1/cuentas/123?campos=id,saldo,moneda
```

| Técnica | Ahorro |
|---|---|
| Compresión gzip/brotli | 60-80% |
| Selección de campos | variable, puede ser alto |
| Paginación con límite razonable | alto |
| Evitar anidamiento profundo | medio |
| Nombres de campo cortos | bajo (no vale la ilegibilidad) |

> [!tip] Comprimir es una línea de configuración con retorno inmediato
> Reduce el costo de egreso, la latencia percibida y el consumo de datos del cliente móvil, todo junto. Es de las primeras cosas que revisaría en cualquier API.

## 🔁 Idempotencia

```java
@PostMapping("/api/v1/transferencias")
public TransferenciaResponse transferir(
        @RequestHeader("Idempotency-Key") @Pattern(regexp = "^[A-Za-z0-9_-]{16,64}$") String clave,
        @Valid @RequestBody TransferenciaRequest request) {
    return idempotencia.ejecutarUnaVez(clave, request, TransferenciaResponse.class,
        () -> servicio.transferir(request));
}
```

| Método | Idempotente por especificación |
|---|---|
| `GET`, `HEAD`, `OPTIONS` | ✅ |
| `PUT`, `DELETE` | ✅ |
| `POST` | ❌ **necesita clave explícita** |
| `PATCH` | depende |

Detalle completo en [[05 - Idempotencia Financiera]].

## ⚡ Operaciones largas

```java
// ❌ El cliente espera 3 minutos; el balanceador corta a los 60 s
@PostMapping("/reportes")
public Reporte generar(@RequestBody Solicitud s) {
    return generador.generar(s);
}

// ✅ Asíncrono con consulta de estado
@PostMapping("/reportes")
@ResponseStatus(HttpStatus.ACCEPTED)                    // 202
public TareaResponse solicitar(@RequestBody Solicitud s) {
    UUID tareaId = cola.encolar(s);
    return new TareaResponse(tareaId, "EN_PROCESO", "/api/v1/tareas/" + tareaId);
}

@GetMapping("/tareas/{id}")
public TareaResponse estado(@PathVariable UUID id) { ... }
```

> [!warning] El timeout del balanceador limita las peticiones síncronas
> Un ALB corta a los 60 segundos por defecto. La solución no es subir el timeout —eso mantiene hilos ocupados y no escala— sino convertir la operación en asíncrona.

## ❌ Errores como contrato

```java
@ExceptionHandler(SaldoInsuficienteException.class)
public ProblemDetail saldoInsuficiente(SaldoInsuficienteException ex) {
    var pd = ProblemDetail.forStatus(HttpStatus.UNPROCESSABLE_ENTITY);
    pd.setType(URI.create("https://corebank/errores/saldo-insuficiente"));
    pd.setTitle("Saldo insuficiente");
    pd.setProperty("disponible", ex.disponible().monto());
    pd.setProperty("solicitado", ex.solicitado().monto());
    return pd;
}
```

| Código | Cuándo |
|---|---|
| `400` | petición mal formada |
| `401` / `403` | no autenticado / sin permiso |
| `404` | no existe |
| `409` | conflicto de estado |
| `422` | bien formada pero inválida por negocio |
| `429` | límite de tasa |
| `503` + `Retry-After` | no disponible temporalmente |

> [!tip] `422` y `400` no son lo mismo
> `400` es "no entiendo tu petición" (JSON roto, tipo incorrecto). `422` es "te entendí perfectamente y no puedo hacerlo" (saldo insuficiente, cuenta bloqueada). La distinción le dice al cliente si tiene sentido reintentar con otros datos.

## 🛡️ Defensas

| Defensa | Por qué |
|---|---|
| Límite de tasa por cliente | [[07 - Rate Limiting]] |
| Tamaño máximo de petición | evita agotar memoria |
| Timeout por endpoint | libera hilos |
| Paginación obligatoria con tope | evita consultas ilimitadas |
| Validación estricta de entrada | seguridad y claridad |
| Compresión | costo y latencia |

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 12MB
server:
  max-http-request-header-size: 16KB
  tomcat:
    max-swallow-size: 10MB
```

> [!danger] Un endpoint sin límite de paginación es un ataque de denegación esperando ocurrir
> `GET /movimientos?limit=1000000` y el servidor intenta materializar un millón de filas. El límite se **impone en el servidor**, no se confía al cliente.

```java
public record ParametrosPagina(Integer limit, String cursor) {
    private static final int MAX = 200;
    private static final int DEFECTO = 50;

    public int limiteEfectivo() {
        return limit == null ? DEFECTO : Math.min(limit, MAX);
    }
}
```

## 🧪 Tests de contrato

```java
@Test
void agregar_un_campo_no_rompe_a_los_clientes_existentes() { ... }

@Test
void el_limite_de_pagina_se_acota_en_el_servidor() {
    var r = api.get("/api/v1/cuentas/1/movimientos?limit=999999");
    assertEquals(200, r.statusCode());
    assertTrue(r.jsonPath().getList("datos").size() <= 200);
}

@Test
void la_paginacion_por_cursor_es_estable_ante_inserciones() {
    var pagina1 = api.get("/movimientos?limit=10");
    insertarMovimientoNuevo();
    var pagina2 = api.get("/movimientos?cursor=" + pagina1.cursor() + "&limit=10");
    assertTrue(interseccionVacia(pagina1.datos(), pagina2.datos()));
}

@Test
void los_errores_siguen_el_formato_ProblemDetail() { ... }
```

Con **Pact** se verifica el contrato entre consumidor y proveedor en CI — ver [[08 - Contract Testing]].

## ⚠️ Errores Comunes

> [!warning] Los que se pagan al escalar
> - **Paginar con `OFFSET`**: costo creciente y resultados inestables.
> - **Sin límite máximo de página**: denegación de servicio trivial.
> - **Sin versión en la ruta** desde el principio.
> - **Cambios que rompen en silencio** (orden por defecto, tipos).
> - **Sin endpoints por lote**: N+1 de red.
> - **Operaciones largas síncronas**: el balanceador las corta.
> - **`POST` sin clave de idempotencia** en operaciones con efecto.
> - **Errores sin formato consistente**.
> - **Sin compresión**.
> - **Exponer entidades de base** como contrato público.

## 📌 Notas Rápidas
- Si tuviera que elegir una sola decisión de esta nota: **paginación por cursor desde el principio**. Es la que más caro sale cambiar después y la que más impacto tiene en tablas grandes.
- El contrato de una API pública es prácticamente permanente: los clientes no actualizan cuando vos querés. Diseñalo pensando que vas a convivir con él años.
- En una entrevista, mencionar cursor vs offset y explicar el problema de estabilidad ante inserciones es una señal clara de experiencia real.
- Se aplica en [[Fase 1 - API de Cuentas]] y [[14 - Open Banking]].

## 🔗 Relaciones
- **Módulo**: [[System Design]]
- **Anterior**: [[02 - Estimaciones]]
- **Siguiente**: [[04 - Estrategias de Cache]]
- **Se conecta con**: [[04 - Diseno de APIs Senior]], [[02 - REST Controllers]], [[11 - gRPC]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

