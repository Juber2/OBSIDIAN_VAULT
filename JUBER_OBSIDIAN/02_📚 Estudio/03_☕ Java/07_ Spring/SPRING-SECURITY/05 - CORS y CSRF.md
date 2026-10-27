---
tags:
  - Java
  - Spring
  - Security
Creado: 2026-05-21
Relacionado:
  - Spring Security
  - REST APIs
Palabras clave:
  - CORS, CSRF, preflight, SameSite, tokens, cross-origin
---

## 🧠 CORS — Cross-Origin Resource Sharing

Un **origen** = protocolo + dominio + puerto.

```
http://localhost:3000  ≠  http://localhost:8080   (diferente puerto)
https://app.com        ≠  https://api.app.com     (diferente subdominio)
http://app.com         ≠  https://app.com          (diferente protocolo)
```

### Por qué existe

El navegador bloquea requests de un origen a otro por seguridad. CORS es el mecanismo que le dice al navegador: "este servidor acepta requests de estos orígenes".

### Preflight request

Para requests con métodos que modifican datos (POST, PUT, DELETE) o headers personalizados, el navegador envía primero un `OPTIONS` para preguntar si está permitido:

```
OPTIONS /api/usuarios
Origin: http://localhost:3000
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type
```

El servidor debe responder:
```
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 3600
```

## Configurar CORS en Spring

### Opción 1: Global en SecurityFilterChain (recomendado)

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .cors(cors -> cors.configurationSource(corsConfigurationSource()))
        .authorizeHttpRequests(...)
        // ...

    return http.build();
}

@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();

    config.setAllowedOrigins(List.of(
        "http://localhost:3000",
        "https://mi-app.com"
    ));

    // O permitir todos los orígenes (solo para desarrollo)
    // config.setAllowedOriginPatterns(List.of("*"));

    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"));

    config.setAllowedHeaders(List.of(
        "Authorization",
        "Content-Type",
        "X-Requested-With"
    ));

    config.setExposedHeaders(List.of("X-Total-Count", "Authorization"));

    config.setAllowCredentials(true);  // para cookies/auth headers

    config.setMaxAge(3600L);  // cache del preflight en segundos

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return source;
}
```

### Opción 2: @CrossOrigin por controller/método

```java
@RestController
@RequestMapping("/api")
@CrossOrigin(origins = "http://localhost:3000")  // aplica a todo el controller
public class ApiController {

    @GetMapping("/datos")
    @CrossOrigin(origins = "*", maxAge = 3600)   // aplica a este método
    public List<Dato> getDatos() { ... }
}
```

### Opción 3: WebMvcConfigurer (sin Spring Security)

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("*")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

> [!warning] Con Spring Security, WebMvcConfigurer no siempre funciona
> Spring Security intercepta antes que Spring MVC. Usar `CorsConfigurationSource` en el `SecurityFilterChain` es más confiable.

## CSRF — Cross-Site Request Forgery

Ataque donde un sitio malicioso hace que el navegador del usuario autenticado envíe requests a tu API (usando las cookies de sesión).

```
Usuario logueado en banco.com
    → visita sitio-malicioso.com
    → sitio-malicioso.com tiene <form action="banco.com/transferir">
    → el navegador envía la request CON las cookies del banco
    → el banco ejecuta la transferencia ❌
```

### Cuándo es relevante

| Escenario | CSRF relevante |
|-----------|---------------|
| Sesión con cookies (web tradicional) | ✅ Sí — habilitar CSRF |
| JWT en header Authorization | ❌ No — los cookies no se envían automáticamente |
| API REST stateless | ❌ No — deshabilitar CSRF |

### Por qué las APIs JWT pueden desactivar CSRF

```
El ataque CSRF funciona porque el navegador envía cookies automáticamente.
Si la autenticación requiere el header "Authorization: Bearer <token>",
el sitio malicioso no puede agregar ese header (same-origin policy).
→ REST APIs con JWT: csrf().disable() es seguro.
```

## Deshabilitar CSRF para REST APIs con JWT

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .csrf(AbstractHttpConfigurer::disable)  // ✅ seguro para JWT stateless
        // ...
    return http.build();
}
```

## CSRF para aplicaciones web con sesión

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .csrf(csrf -> csrf
            .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            // Token en cookie legible por JS (para SPAs)
        )
        // ...
    return http.build();
}
```

```javascript
// Frontend: leer token de la cookie y enviarlo en header
const csrfToken = getCookie('XSRF-TOKEN');
fetch('/api/datos', {
    method: 'POST',
    headers: {
        'X-XSRF-TOKEN': csrfToken,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(datos)
});
```

## SameSite cookies — alternativa moderna al CSRF

```java
// Spring Boot — configurar cookies con SameSite
server:
  servlet:
    session:
      cookie:
        same-site: strict  # o 'lax'
        secure: true        # solo HTTPS
        http-only: true     # JS no puede leer
```

| SameSite | Comportamiento |
|----------|---------------|
| `Strict` | Nunca enviar cookie cross-site |
| `Lax` | Solo GET cross-site (links normales) |
| `None` | Siempre enviar (requiere Secure) |

## Headers de seguridad adicionales

Spring Security los agrega por defecto:

```java
// Ya incluidos automáticamente:
// X-Content-Type-Options: nosniff
// X-Frame-Options: DENY
// X-XSS-Protection: 1; mode=block
// Cache-Control: no-cache, no-store, max-age=0, must-revalidate
// Content-Security-Policy: (si lo configuras)

@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.headers(headers -> headers
        .frameOptions(frame -> frame.deny())
        .contentTypeOptions(Customizer.withDefaults())
        .contentSecurityPolicy(csp -> csp
            .policyDirectives("default-src 'self'; script-src 'self'")
        )
        .httpStrictTransportSecurity(hsts -> hsts
            .includeSubDomains(true)
            .maxAgeInSeconds(31536000)
        )
    );

    return http.build();
}
```

## Checklist de seguridad para REST APIs

```
✅ SessionCreationPolicy.STATELESS
✅ csrf().disable() (JWT stateless)
✅ CORS configurado con orígenes específicos (no "*" en producción)
✅ HTTPS en producción
✅ JWT con expiración corta (15min-1h)
✅ Refresh tokens rotados
✅ Contraseñas con BCrypt strength >= 10
✅ No exponer stack traces en errores
✅ Rate limiting en endpoints de auth
✅ Audit log de eventos de seguridad
```

## 📌 Notas Rápidas
- CORS es una protección del **navegador**, no del servidor — un cliente no-browser (Postman, otro backend) nunca se ve afectado por configuración CORS, es fácil confundir esto en entrevista.
- El preflight `OPTIONS` solo se dispara para requests "no simples" (con métodos como PUT/DELETE, o headers custom como `Authorization`) — un GET simple sin headers extra no lo dispara.
- CSRF y CORS resuelven problemas distintos: CORS controla **qué orígenes pueden leer** la respuesta; CSRF previene que un sitio ajeno **dispare acciones** aprovechando cookies de sesión ya autenticadas.
- Nunca usar `allowedOrigins("*")` junto con `allowCredentials(true)` — el estándar CORS directamente lo prohíbe (riesgo de seguridad), hay que listar orígenes explícitos.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

