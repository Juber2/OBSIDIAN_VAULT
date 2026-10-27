---
tags:
  - Azure
  - Seguridad
  - EntraID
  - OAuth2
Creado: 2026-08-30
Relacionado:
  - Azure
  - Laboratorios
---

# Entra ID y Seguridad de APIs

> Pregunta literal de la vacante de Entelgy: **"¿Cuál es la diferencia entre OAuth2, OIDC y JWT?"** y **"¿Cómo proteges una API con Entra ID?"**. Seguridad aparece en 7 de 14 vacantes del radar. Entra ID es el antiguo Azure Active Directory (AAD) — los anuncios de trabajo usan ambos nombres.

## OAuth 2.0 vs OIDC vs JWT — la pregunta que más se falla

Se confunden porque siempre aparecen juntos, pero responden cosas distintas:

| | Qué es | Responde a |
|---|---|---|
| **OAuth 2.0** | Framework de **autorización** | "¿Esta aplicación puede acceder a este recurso en nombre de alguien?" |
| **OIDC** | Capa de **identidad** construida sobre OAuth 2.0 | "¿Quién es este usuario?" |
| **JWT** | **Formato** de token (JSON firmado en Base64) | Nada — es el sobre, no el contenido conceptual |

La forma corta y correcta de responderlo:

> *"OAuth 2.0 es autorización, OIDC agrega autenticación encima de OAuth, y JWT es solo el formato en que suele viajar el token. Se pueden usar tokens opacos en vez de JWT y seguiría siendo OAuth 2.0."*

Ese último matiz —que JWT es intercambiable— es lo que demuestra que entendiste la diferencia en vez de memorizar tres siglas.

### Los dos tokens que emite OIDC

- **ID Token**: dice *quién es el usuario*. Lo consume la **aplicación cliente**, nunca la API.
- **Access Token**: dice *a qué tiene permiso*. Es el que viaja a la API en `Authorization: Bearer`.

Error clásico: mandar el ID Token a la API. La API debe rechazarlo — su `audience` no le corresponde.

## App Registration: los dos registros

Para proteger una API con Entra ID hay **dos** registros distintos, y confundirlos es fuente habitual de errores:

```
[App Registration: mi-api]        ← expone permisos (scopes/roles)
        ▲
        │ pide un token para acceder a la API
        │
[App Registration: mi-frontend]   ← consume permisos
```

```bash
# Registrar la API
az ad app create --display-name "mi-api" --sign-in-audience AzureADMyOrg

# Exponerla con un Application ID URI
az ad app update --id <app-id> --identifier-uris "api://mi-api"
```

## Scopes vs App Roles — delegated vs application

Esta distinción aparece seguido en entrevistas de Azure:

| | Delegated permissions (scopes) | Application permissions (app roles) |
|---|---|---|
| Quién actúa | La app **en nombre de un usuario** | La app **por sí misma** |
| Hay usuario presente | Sí | No |
| Flujo típico | Authorization Code + PKCE | Client Credentials |
| Claim en el token | `scp` | `roles` |
| Ejemplo | Una SPA lee las órdenes *del usuario logueado* | Un job nocturno concilia *todas* las órdenes |

**El permiso efectivo del delegado es la intersección** de lo que la app puede hacer y lo que el usuario puede hacer — aunque la app tenga el scope, si el usuario no tiene el permiso de negocio, no debe pasar. Por eso la autorización por recurso sigue siendo necesaria en la API (A01 de OWASP, ver [[03 - Seguridad Backend Java]]).

## Flujo 1 — Usuario: Authorization Code + PKCE

```
1. SPA redirige al usuario a Entra ID
2. Usuario se autentica (MFA si aplica)
3. Entra ID devuelve un CODE (no un token) a la SPA
4. La SPA canjea el code + code_verifier por tokens
5. La SPA llama a la API con el Access Token
```

**Por qué PKCE:** una SPA o app móvil no puede guardar un client secret (el código es público). PKCE reemplaza el secreto por un desafío criptográfico generado en cada login, de modo que interceptar el `code` no alcanza para canjearlo.

> Hoy PKCE se recomienda **también** para clientes confidenciales. El flujo Implicit está deprecado — mencionarlo como opción válida es una señal de conocimiento desactualizado.

## Flujo 2 — Servicio a servicio: Client Credentials

```
1. El servicio pide un token a Entra ID con su client_id + credencial
2. Entra ID devuelve un Access Token (sin usuario involucrado)
3. El servicio llama a la API destino con ese token
```

```bash
az ad app credential reset --id <app-id> --years 1
```

Pero la mejor versión de este flujo **no usa credencial en absoluto**: si el servicio corre en Azure, usa **Managed Identity** y no hay secreto que rotar ni filtrar (ver [[01 - Key Vault y Managed Identity]]). Cuando corre fuera de Azure —por ejemplo un pipeline de GitHub Actions— la alternativa correcta es **federated credentials (OIDC)**, no un client secret de larga duración.

## Validar el token en la API

Un token es de confianza solo si validás **las cuatro cosas**:

| Qué validar | Por qué |
|---|---|
| **Firma** | Que lo emitió realmente Entra ID y nadie lo alteró |
| **Issuer** (`iss`) | Que viene de tu tenant, no de otro directorio |
| **Audience** (`aud`) | Que fue emitido **para tu API**, no para otra |
| **Expiración** (`exp`) | Que sigue vigente |

Omitir la validación de `audience` es una vulnerabilidad real: un token válido emitido para *otra* aplicación sería aceptado por tu API.

### .NET

```csharp
builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("LeerOrdenes", policy =>
        policy.RequireClaim("scp", "Ordenes.Read"));
});
```

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "<tenant-id>",
    "ClientId": "<api-client-id>",
    "Audience": "api://mi-api"
  }
}
```

```csharp
[Authorize(Policy = "LeerOrdenes")]
[HttpGet("{id}")]
public async Task<IActionResult> Get(int id)
{
    var orden = await _repo.ObtenerAsync(id);

    // Autorización por RECURSO: tener el scope no alcanza
    if (orden.ClienteId != User.GetObjectId())
        return Forbid();

    return Ok(orden);
}
```

Ese segundo chequeo es el punto crítico: **el scope dice que la app puede leer órdenes; no dice que este usuario pueda leer *esta* orden.**

### Java/Spring

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
        return http.build();
    }
}
```

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://login.microsoftonline.com/<tenant-id>/v2.0
          audiences: api://mi-api
```

```java
@PreAuthorize("hasAuthority('SCOPE_Ordenes.Read')")
@GetMapping("/{id}")
public OrdenDto obtener(@PathVariable Long id, @AuthenticationPrincipal Jwt jwt) {
    Orden orden = repo.findById(id).orElseThrow();
    if (!orden.getClienteId().equals(jwt.getSubject())) {
        throw new AccessDeniedException("Forbidden");
    }
    return mapper.toDto(orden);
}
```

## Dónde encaja APIM

Si hay API Management delante (ver [[04 - APIM y Application Gateway]]), APIM puede validar el JWT antes de que el tráfico llegue a la API — útil para rate limiting y para rechazar tokens inválidos en el borde.

**Pero la API no debe confiar solo en eso.** Si alguien alcanza la API por otra ruta (red interna, otro servicio), la validación del gateway no la protege. Defensa en profundidad: validar en ambos lados.

## Errores comunes

- Decir que "OAuth2 es autenticación" — es autorización; la autenticación la agrega OIDC.
- Mandar el ID Token a la API en vez del Access Token.
- No validar `audience`: aceptás tokens legítimos emitidos para otra aplicación.
- Confiar solo en el scope y no verificar ownership del recurso (A01 Broken Access Control).
- Usar client secrets de larga duración donde correspondía Managed Identity o federated credentials.
- Proponer el flujo Implicit, deprecado hace años.
- Validar el token únicamente en APIM y dejar la API abierta por dentro.

## Checklist

- [ ] Puedo explicar OAuth2 vs OIDC vs JWT en tres frases, incluyendo que JWT es intercambiable.
- [ ] Sé la diferencia entre ID Token y Access Token y quién consume cada uno.
- [ ] Sé distinguir delegated (scopes, claim `scp`) de application (app roles, claim `roles`).
- [ ] Puedo explicar qué problema resuelve PKCE y por qué Implicit está deprecado.
- [ ] Sé enumerar las cuatro validaciones de un token y qué pasa si falta `audience`.
- [ ] Sé por qué el scope no reemplaza la autorización por recurso.
- [ ] Sé cuándo usar Managed Identity en lugar de client credentials.
