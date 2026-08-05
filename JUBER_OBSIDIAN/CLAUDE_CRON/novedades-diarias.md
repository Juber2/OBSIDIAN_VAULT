# Novedades Diarias

Rotación de 4 secciones, en ciclo continuo según la última entrada agregada: Noticias Tech → Tip de Programación → Repaso AZ-204 → Inglés Técnico → (repite).

---

## 2026-07-25 12:43 — Repaso AZ-204

**Pregunta:** Tu aplicación usa Azure Cosmos DB para un carrito de compras global (múltiples regiones de escritura). Necesitas garantizar que las lecturas nunca vean escrituras fuera de orden dentro de una misma sesión de cliente, pero puedes tolerar algo de latencia adicional entre regiones distintas. ¿Qué nivel de consistencia deberías configurar?

A) Strong
B) Bounded Staleness
C) Session
D) Eventual

**Respuesta: C) Session**

**Explicación:** Session consistency garantiza monotonic reads, monotonic writes y "read-your-writes" dentro del mismo *session token* (mismo cliente/sesión), sin pagar el costo de latencia y disponibilidad de Strong (que requiere consenso síncrono entre todas las réplicas). Bounded Staleness acota el desfase por tiempo o versiones pero no da la garantía "read-your-writes" tan barata como Session. Eventual no da ningún orden garantizado. Session es el default de Cosmos DB y el más usado en apps multi-región porque balancea consistencia percibida por usuario vs. throughput/latencia.

---

## 2026-07-25 17:05 — Inglés Técnico

1. **Flaky test** — prueba que falla de forma intermitente sin cambios en el código.
   *"This test is flaky, it fails 1 in 10 runs — let's add a retry or fix the race condition."*

2. **Nitpick** — comentario menor, de estilo, no bloqueante.
   *"Just a nitpick: rename this variable to be more descriptive, doesn't block the merge."*

3. **Footgun** — patrón o API fácil de usar mal, que termina causando bugs.
   *"Exposing a public setter here is a footgun — someone will mutate state outside the reducer."*

4. **Blast radius** — alcance del impacto si algo falla.
   *"This change touches the shared auth middleware, so the blast radius is the whole API, not just this endpoint."*

5. **Bikeshedding** — discutir demasiado detalles triviales mientras se ignoran los importantes.
   *"Let's not bikeshed the naming convention here — approve the PR and raise it in the next tech sync."*

---

## 2026-07-25 23:10 — Noticias Tech

- **.NET**: Microsoft publicó los servicing releases de julio 2026 (.NET 10.0.10, 9.0.18 y 8.0.29) con fixes de seguridad y no-seguridad. Además, .NET 11 Preview 6 ya está disponible, y .NET MAUI pasa a ser CoreCLR-only en Android, iOS y Mac Catalyst.
- **Azure**: incidente regional en West US el 23 de julio (14:44–19:41 UTC) con fallas de conectividad y latencia elevada; servicio restaurado hacia las 18:26 UTC.
- **Angular**: se liberó Angular 22.0.8 (22 de julio) junto con el RC 22.1.0-rc.0. Angular consolida su "era signal-first" (Signal Forms estables) y hay un PR en curso para cambiar el ciclo de releases a uno anual en vez de semestral.

---

## 2026-07-26 13:40 — Tip de Programación

**EF Core `ExecuteUpdate`/`ExecuteDelete` rompen tus eventos de dominio.**

En Clean Architecture, si disparas *domain events* desde `SaveChanges` (interceptor o override), los bulk operations `ExecuteUpdate`/`ExecuteDelete` (EF Core 7+) los saltan por completo: no cargan entidades, no pasan por el `ChangeTracker`, no llaman a `SaveChangesAsync`. Son directos a SQL. Útil para performance en un `UPDATE` masivo, pero si un Command Handler los usa "porque es más rápido", pierdes silenciosamente cualquier efecto secundario (notificaciones, invalidación de caché, outbox, etc.) que dependa de esos eventos.

```csharp
// Traiciona el patrón: no dispara OrderStatusChangedEvent
await dbContext.Orders
    .Where(o => o.WarehouseId == warehouseId)
    .ExecuteUpdateAsync(s => s.SetProperty(o => o.Status, OrderStatus.Shipped));

// Correcto si necesitas el evento: cargar, mutar, SaveChanges
var orders = await dbContext.Orders
    .Where(o => o.WarehouseId == warehouseId)
    .ToListAsync();

foreach (var o in orders)
    o.MarkAsShipped(); // agrega el domain event internamente

await dbContext.SaveChangesAsync();
```

Regla práctica: usa `ExecuteUpdate`/`ExecuteDelete` solo en Queries/reportes o mutaciones puramente técnicas sin reglas de negocio asociadas; para Commands que deben emitir eventos de dominio, sigue el camino tradicional de tracking + `SaveChanges`.

---

## 2026-07-26 16:13 — Repaso AZ-204

**Pregunta:** Estás escribiendo una *orchestrator function* de Azure Durable Functions que debe generar un identificador único para una nueva orden y registrar la hora actual en el historial. Un desarrollador propone usar `Guid.NewGuid()` y `DateTime.UtcNow` directamente dentro del código del orchestrator. ¿Por qué es esto problemático y cuál es la solución correcta?

A) No es problemático, ambas llamadas son thread-safe.
B) Es problemático porque el orchestrator se re-ejecuta (replay) desde el historial, y estas llamadas no son deterministas; se debe usar `context.NewGuid()` y `context.CurrentUtcDateTime`.
C) Es problemático solo por rendimiento; conviene cachear los valores en una variable estática.
D) Es problemático porque `Guid.NewGuid()` no está disponible en Azure Functions; hay que generarlo en una Activity Function.

**Respuesta: B**

**Explicación:** El motor de Durable Functions reconstruye el estado del orchestrator reproduciendo (*replaying*) su historial de eventos cada vez que se despierta (tras un `await`, un checkpoint, etc.). Para que el replay produzca el mismo resultado siempre, el código del orchestrator debe ser determinista: nada de `DateTime.UtcNow`, `Guid.NewGuid()`, llamadas de red, lecturas de archivo o generación de números aleatorios directamente. El framework expone equivalentes deterministas —`context.CurrentUtcDateTime` y `context.NewGuid()`— que quedan registrados en el historial y devuelven el mismo valor en cada replay. La opción D es una alternativa técnicamente posible pero innecesaria: el problema no es dónde se genera el GUID, sino usar la API no determinista dentro del orchestrator.

---

## 2026-07-26 22:05 — Inglés Técnico

1. **Rubber-stamp** — aprobar un PR sin revisarlo realmente, como trámite.
   *"Don't just rubber-stamp this — it touches the payment flow, take a real look."*

2. **Bus factor** — cuántas personas pueden desaparecer del equipo antes de que el conocimiento sobre algo se pierda.
   *"This module has a bus factor of one; let's pair on it so someone else understands the retry logic."*

3. **Shotgun surgery** — un cambio pequeño que obliga a tocar muchos archivos/módulos distintos (mal síntoma de diseño).
   *"Adding a field here is shotgun surgery — it touches five layers, we should consolidate that mapping first."*

4. **Happy path** — el flujo principal donde todo sale bien, sin errores ni casos borde.
   *"The happy path looks fine, but what happens if the token expires mid-request?"*

5. **Churn** — qué tanto y con qué frecuencia cambia un archivo/código (indicador de inestabilidad o riesgo).
   *"This file has high churn and low test coverage — that combo is where most of our bugs come from."*

---

## 2026-07-27 13:52 — Noticias Tech

- **.NET**: la redefinición de la palabra clave `unsafe` en .NET 11 (ya en preview) pasa de ser solo sintaxis a marcar contratos explícitos entre caller/callee, haciendo visibles y revisables las obligaciones de seguridad — motivado en parte por el auge del código generado por IA. Además, .NET 8 y .NET 9 llegan a fin de soporte el 10 de noviembre de 2026; toca planear el upgrade a .NET 10 (LTS).
- **Azure**: nuevo preview público de **Chaos Studio Workspaces y Scenarios**, con soporte directo desde Azure CLI (un solo comando aprovisiona workspace y configuración) para pruebas de resiliencia más rápidas y con menor latencia de simulación de outages.
- **Azure**: preview temprano de **VMs Cobalt 200 (Arm)** orientadas a cargas de IA agentic sobre Linux.

---

## 2026-07-27 23:10 — Tip de Programación

**`OwnsMany` en EF Core: reemplazar un ítem de la colección "in situ" no se guarda.**

Cuando modelas una colección de *value objects* como `OwnsMany` (owned types), EF Core rastrea la colección completa como parte de la entidad dueña, pero **no detecta reemplazos de referencia dentro de un elemento individual** de forma confiable si solo mutas una propiedad anidada a través de una referencia vieja obtenida antes de un `SaveChanges`, o si reconstruyes el objeto con `with` (record) y lo vuelves a insertar en la lista sin pasar por los métodos del agregado. El síntoma clásico: el `UPDATE` nunca llega a la base de datos y no hay ninguna excepción — simplemente no pasa nada.

```csharp
// Mal: reemplazas el objeto pero EF no siempre detecta el cambio
// si la lista se obtuvo de un contexto distinto o el tracking se perdió
var direccion = pedido.Direcciones.First(d => d.Tipo == TipoDireccion.Envio);
direccion = direccion with { Calle = "Nueva 123" }; // nueva instancia "suelta"

// Bien: el Aggregate Root controla el reemplazo explícitamente
public void ActualizarDireccionEnvio(Direccion nueva)
{
    var actual = _direcciones.Single(d => d.Tipo == TipoDireccion.Envio);
    _direcciones.Remove(actual);
    _direcciones.Add(nueva with { Tipo = TipoDireccion.Envio });
    // EF detecta remove+add en la colección owned sin ambigüedad
}
```

Regla práctica: nunca expongas la colección `OwnsMany` como mutable desde fuera del agregado; que el propio Aggregate Root haga `Remove` + `Add` explícito en vez de intentar mutar o reasignar un elemento por referencia — así el `ChangeTracker` no tiene ambigüedad sobre qué cambió.

---

## 2026-07-27 17:05 — Repaso AZ-204

**Pregunta:** Tu App Service necesita leer un secreto desde Azure Key Vault en tiempo de ejecución. La política de la empresa prohíbe almacenar credenciales o cadenas de conexión en el código o en la configuración de la app. ¿Cuál es la forma recomendada de dar acceso al App Service para leer ese secreto?

A) Crear un Service Principal, guardar su `client secret` en Application Settings y usarlo para autenticar contra Key Vault.
B) Habilitar una **Managed Identity** (system-assigned o user-assigned) en el App Service y asignarle una política de acceso (o rol RBAC) sobre el Key Vault.
C) Usar la cuenta de administrador de la suscripción para generar un token de acceso y guardarlo cifrado en una variable de entorno.
D) Descargar el secreto una vez y guardarlo como App Setting cifrado con Azure Storage Service Encryption.

**Respuesta: B) Managed Identity**

**Explicación:** Managed Identity le da al App Service una identidad en Azure AD gestionada automáticamente por la plataforma (sin que el desarrollador maneje ni rote ningún secreto). Con esa identidad, se le otorga acceso a Key Vault vía access policy o, en el modelo más moderno, vía rol RBAC (`Key Vault Secrets User`). En runtime, el SDK (`DefaultAzureCredential`) obtiene el token automáticamente. Las opciones A, C y D reintroducen exactamente el problema que se quiere evitar: un secreto (client secret, token o el propio valor) que alguien tiene que almacenar, rotar y proteger manualmente. Managed Identity es la respuesta estándar de AZ-204 para "cómo autenticar un recurso de Azure contra otro sin manejar credenciales".

---

## 2026-07-28 05:26 — Inglés Técnico

1. **Code smell** — señal de que algo está mal diseñado, aunque no sea un bug en sí.
   *"This class has a code smell — 15 constructor params usually means it's doing too much."*

2. **Yak shaving** — cadena de tareas previas, cada vez más alejadas del objetivo original, antes de poder hacer el cambio real.
   *"I wanted to fix the bug but ended up yak shaving — updating the SDK, then the CI image, then the Dockerfile."*

3. **Cargo cult programming** — copiar un patrón o código sin entender por qué funciona, solo porque "así se hace".
   *"Adding `[ConfigureAwait(false)]` everywhere without knowing why is cargo cult programming — it doesn't apply the same way in ASP.NET Core."*

4. **Leaky abstraction** — una abstracción que no oculta del todo los detalles internos, y estos se filtran al consumidor.
   *"Returning `IQueryable` from the repository is a leaky abstraction — callers end up depending on EF Core's translation quirks."*

5. **Gold plating** — agregar funcionalidad o pulido no solicitado, más allá de lo que el requerimiento pedía.
   *"Let's not gold plate this endpoint with caching and retries nobody asked for — ship the simple version first."*

---

## 2026-07-28 11:15 — Noticias Tech

- **Angular**: se confirma Angular 22.0.8 como release estable de julio 2026 (soporte activo hasta jun-2028), con el RC 22.1.0-rc.0 ya circulando de cara a la próxima minor.
- **.NET / Azure**: la búsqueda web no estuvo disponible en este ciclo (límite de uso mensual alcanzado), así que no se pudo verificar novedades propias de las últimas 24h para estos dos. Se retoma en el próximo ciclo de Noticias Tech.

---

