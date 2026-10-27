---
tags:
  - Java
  - Proyecto
  - Banca
  - corebank
  - Seguridad
Creado: 2026-09-08
Relacionado:
  - Proyectos Java
  - Spring Security
Palabras clave:
  - JWT, refresh token, RBAC, autorizacion por recurso, auditoria, BCrypt, append-only
Estado:
  - Pendiente
Nivel: Intermedio-Avanzado
Prerequisitos: Fase 2, Spring Security
---

# Fase 3 · Seguridad y auditoría

## 🎯 Objetivo

Hasta ahora cualquiera con acceso a la red puede transferir el dinero de cualquiera. En esta fase `corebank` aprende **quién sos, qué podés hacer y sobre qué**, y empieza a registrar de forma inmutable todo lo que pasó.

## 🧠 Las tres preguntas que hay que separar

| Pregunta | Nombre | Ejemplo |
|---|---|---|
| ¿Quién sos? | **Autenticación** | el token es válido y corresponde a Juber |
| ¿Qué podés hacer? | **Autorización por rol** | los operadores pueden reversar, los clientes no |
| ¿Sobre qué? | **Autorización por recurso** | Juber puede ver *sus* cuentas, no las de otro |

> [!danger] La tercera es la que se olvida
> Es el fallo **IDOR** (referencia directa insegura a objeto), y encabeza el OWASP Top 10. El endpoint valida que el token sea válido y que el rol sea `CLIENTE`… y después devuelve la cuenta que venga en la URL. Cambiar el UUID en la petición te muestra las cuentas de otro cliente.

## 🔑 Modelo de acceso

```
Usuario ──► Rol ──► Permisos

CLIENTE   · ver sus cuentas · ver sus movimientos · transferir desde sus cuentas
OPERADOR  · ver cualquier cuenta · registrar depósitos y retiros · reversar
AUDITOR   · ver todo · exportar auditoría · NO puede mover dinero
ADMIN     · gestionar usuarios y roles · NO puede mover dinero
```

> [!tip] Separación de funciones
> Que `ADMIN` y `AUDITOR` **no puedan mover dinero** no es un olvido: es un control obligatorio en banca. Quien administra los accesos no debe poder operar, y quien audita tampoco. Si un solo rol puede hacer todo, no hay control real.

## 🗄️ Esquema

```sql
-- V3__seguridad_y_auditoria.sql

CREATE TABLE usuario (
    id              UUID PRIMARY KEY,
    username        VARCHAR(60)  NOT NULL UNIQUE,
    password_hash   VARCHAR(120) NOT NULL,       -- BCrypt, nunca el password
    cliente_id      UUID         REFERENCES cliente(id),   -- null para staff
    activo          BOOLEAN      NOT NULL DEFAULT true,
    intentos_fallidos SMALLINT   NOT NULL DEFAULT 0,
    bloqueado_hasta TIMESTAMPTZ,
    creado_en       TIMESTAMPTZ  NOT NULL DEFAULT now()
);

CREATE TABLE rol (
    id      SMALLINT PRIMARY KEY,
    nombre  VARCHAR(30) NOT NULL UNIQUE
);

CREATE TABLE usuario_rol (
    usuario_id UUID     NOT NULL REFERENCES usuario(id),
    rol_id     SMALLINT NOT NULL REFERENCES rol(id),
    PRIMARY KEY (usuario_id, rol_id)
);

CREATE TABLE refresh_token (
    id            UUID PRIMARY KEY,
    usuario_id    UUID        NOT NULL REFERENCES usuario(id),
    token_hash    VARCHAR(64) NOT NULL UNIQUE,   -- hash, no el token
    expira_en     TIMESTAMPTZ NOT NULL,
    revocado_en   TIMESTAMPTZ,
    reemplazado_por UUID
);

-- Bitácora append-only: sin UPDATE ni DELETE, nunca
CREATE TABLE auditoria (
    id            BIGSERIAL PRIMARY KEY,
    ocurrido_en   TIMESTAMPTZ  NOT NULL DEFAULT now(),
    usuario_id    UUID,
    username      VARCHAR(60),          -- desnormalizado a propósito
    accion        VARCHAR(60)  NOT NULL,
    recurso_tipo  VARCHAR(40),
    recurso_id    VARCHAR(60),
    resultado     VARCHAR(20)  NOT NULL,  -- OK | RECHAZADO | ERROR
    ip            INET,
    trace_id      VARCHAR(40),
    detalle       JSONB
);

CREATE INDEX idx_auditoria_fecha   ON auditoria(ocurrido_en DESC);
CREATE INDEX idx_auditoria_usuario ON auditoria(usuario_id, ocurrido_en DESC);
CREATE INDEX idx_auditoria_recurso ON auditoria(recurso_tipo, recurso_id);
```

```sql
-- La inmutabilidad se impone en la base, no se confía al código
REVOKE UPDATE, DELETE ON auditoria FROM corebank_app;

CREATE RULE auditoria_sin_update AS ON UPDATE TO auditoria DO INSTEAD NOTHING;
CREATE RULE auditoria_sin_delete AS ON DELETE TO auditoria DO INSTEAD NOTHING;
```

> [!tip] Por qué se guarda `username` además de `usuario_id`
> Si mañana se borra o renombra el usuario, la bitácora tiene que seguir diciendo quién fue. La auditoría se desnormaliza a propósito: es una foto del momento, no una vista viva.

## 💻 Piezas clave

### Configuración de seguridad

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity          // habilita @PreAuthorize
public class SeguridadConfig {

    @Bean
    SecurityFilterChain filtros(HttpSecurity http, FiltroJwt filtroJwt) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)          // API sin cookies: no aplica
            .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/swagger/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated())
            .addFilterBefore(filtroJwt, UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(e -> e
                .authenticationEntryPoint(this::sin401Filtrado)
                .accessDeniedHandler(this::sin403Filtrado))
            .build();
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);   // coste 12: ~250ms por hash
    }
}
```

### Autorización por recurso — lo importante

```java
@Component("accesoCuenta")
public class EvaluadorAccesoCuenta {

    private final CuentaRepository cuentas;

    /** ¿El usuario autenticado puede tocar esta cuenta? */
    public boolean puedeAcceder(UUID cuentaId, Authentication auth) {
        var principal = (UsuarioAutenticado) auth.getPrincipal();

        // Staff ve todo
        if (principal.tieneAlgunRol("OPERADOR", "AUDITOR")) return true;

        // Un cliente solo sus propias cuentas
        return cuentas.findById(cuentaId)
            .map(c -> c.getClienteId().equals(principal.clienteId()))
            .orElse(false);
    }
}
```

```java
@GetMapping("/api/v1/cuentas/{id}")
@PreAuthorize("@accesoCuenta.puedeAcceder(#id, authentication)")
public CuentaResponse obtener(@PathVariable UUID id) { ... }

@PostMapping("/api/v1/transferencias")
@PreAuthorize("@accesoCuenta.puedeAcceder(#request.origen(), authentication)")
public TransferenciaResponse transferir(@Valid @RequestBody TransferenciaRequest request) { ... }

@PostMapping("/api/v1/asientos/{id}/reversas")
@PreAuthorize("hasRole('OPERADOR')")
public ReversaResponse reversar(@PathVariable UUID id, @RequestBody ReversaRequest r) { ... }
```

> [!warning] La validación va sobre la cuenta **origen**
> En una transferencia, lo que hay que autorizar es de dónde sale el dinero. Validar el destino no protege nada: cualquiera puede recibir.

### JWT

```java
@Service
public class ServicioJwt {

    private static final Duration VIDA_ACCESS  = Duration.ofMinutes(15);
    private static final Duration VIDA_REFRESH = Duration.ofDays(7);

    public String generarAccessToken(UsuarioAutenticado usuario) {
        Instant ahora = Instant.now();
        return Jwts.builder()
            .subject(usuario.id().toString())
            .claim("username", usuario.username())
            .claim("roles",    usuario.roles())
            .claim("clienteId", usuario.clienteId())
            .issuedAt(Date.from(ahora))
            .expiration(Date.from(ahora.plus(VIDA_ACCESS)))
            .issuer("corebank")
            .signWith(clave, Jwts.SIG.HS256)
            .compact();
    }
}
```

| Token | Vida | Dónde vive | Se puede revocar |
|---|---|---|---|
| Access | 15 min | memoria del cliente | no (por eso dura poco) |
| Refresh | 7 días | almacenamiento seguro | sí, está en la base |

> [!danger] Nunca metas datos sensibles en el JWT
> El payload va en Base64, **no está cifrado**: cualquiera lo lee pegándolo en jwt.io. Saldos, documento de identidad, correo — nada de eso va adentro. Solo identificadores y roles.

### Rotación de refresh tokens

```java
@Transactional
public TokensResponse refrescar(String refreshToken) {
    String hash = sha256(refreshToken);

    RefreshTokenEntity actual = refreshRepo.findByTokenHash(hash)
        .orElseThrow(() -> new TokenInvalidoException());

    // Reuso de un token ya rotado = señal de robo → se corta toda la cadena
    if (actual.getRevocadoEn() != null) {
        refreshRepo.revocarTodosDelUsuario(actual.getUsuarioId());
        auditoria.registrar(SOSPECHA_ROBO_TOKEN, actual.getUsuarioId());
        throw new TokenReutilizadoException();
    }

    if (actual.getExpiraEn().isBefore(Instant.now())) {
        throw new TokenExpiradoException();
    }

    RefreshTokenEntity nuevo = crearRefreshToken(actual.getUsuarioId());
    actual.revocar(nuevo.getId());
    return new TokensResponse(generarAccessToken(...), nuevo.getToken());
}
```

### Auditoría automática

```java
@Aspect
@Component
public class AspectoAuditoria {

    @AfterReturning(pointcut = "@annotation(auditable)", returning = "resultado")
    public void registrarExito(JoinPoint jp, Auditable auditable, Object resultado) {
        auditoria.registrar(EventoAuditoria.builder()
            .accion(auditable.accion())
            .recursoTipo(auditable.recurso())
            .recursoId(extraerId(jp, resultado))
            .usuario(usuarioActual())
            .resultado(OK)
            .ip(ipActual())
            .traceId(MDC.get("traceId"))
            .build());
    }

    @AfterThrowing(pointcut = "@annotation(auditable)", throwing = "ex")
    public void registrarFallo(JoinPoint jp, Auditable auditable, Throwable ex) {
        // Los intentos rechazados se auditan IGUAL que los exitosos
        auditoria.registrar(EventoAuditoria.builder()
            .accion(auditable.accion())
            .usuario(usuarioActual())
            .resultado(ex instanceof AccessDeniedException ? RECHAZADO : ERROR)
            .detalle(Map.of("error", ex.getClass().getSimpleName()))
            .build());
    }
}
```

```java
@Auditable(accion = "TRANSFERENCIA", recurso = "CUENTA")
public TransferenciaResponse transferir(TransferenciaRequest req) { ... }
```

> [!tip] Auditá también lo que falla
> Diez intentos rechazados seguidos sobre cuentas ajenas es la señal más clara de un ataque. Si solo registrás los éxitos, ese patrón es invisible.

### Qué se audita

| Acción | Se audita |
|---|---|
| Login exitoso / fallido | ✅ |
| Bloqueo de cuenta por intentos | ✅ |
| Transferencia, depósito, retiro | ✅ |
| Reversa (con motivo y operador) | ✅ |
| Acceso denegado a un recurso ajeno | ✅ |
| Alta o cambio de rol de un usuario | ✅ |
| Consulta de saldo | ❌ *(ruido: son millones)* |

## ✅ Criterios de aceptación

- [ ] Todos los endpoints salvo login, health y Swagger exigen autenticación.
- [ ] Un cliente **no puede** ver ni operar cuentas de otro (test explícito de IDOR).
- [ ] `AUDITOR` y `ADMIN` no pueden mover dinero.
- [ ] Las contraseñas se guardan con BCrypt; ninguna en claro ni en logs.
- [ ] El access token dura ≤ 15 min y el refresh rota en cada uso.
- [ ] Reusar un refresh token revocado invalida toda la cadena del usuario.
- [ ] La tabla `auditoria` rechaza `UPDATE` y `DELETE` a nivel de base.
- [ ] Se auditan éxitos **y** rechazos, con usuario, IP y trace id.
- [ ] Tras 5 intentos fallidos la cuenta se bloquea temporalmente.
- [ ] Ningún dato sensible viaja dentro del JWT.
- [ ] Los errores de auth devuelven `401`/`403` sin revelar si el usuario existe.

## ⚠️ Trampas de esta fase

> [!warning] Las que se ven en auditorías reales
> - **Validar el rol pero no el dueño del recurso.** Es el IDOR: rol correcto, cuenta ajena.
> - **"Usuario o contraseña incorrectos" vs "usuario no existe".** El segundo mensaje permite enumerar usuarios. Siempre el mismo texto y el mismo tiempo de respuesta.
> - **Guardar el refresh token en claro.** Si se filtra la base, se filtran todas las sesiones. Guardá el hash.
> - **Loguear el request completo.** El body del login lleva la contraseña; si va al log, quedó en texto plano para siempre.
> - **Auditar desde el servicio a mano.** Se olvida en la mitad de los métodos. Con aspecto o interceptor es sistemático.

## 🔁 Retrospectiva

- ¿Por qué el access token dura 15 minutos y no 8 horas? ¿Qué se gana y qué se pierde?
- ¿Qué pasa si te roban un refresh token? ¿Cómo se entera el sistema?
- ¿Por qué la auditoría no debe poder editarse **ni siquiera por un admin**?
- Si tuvieras que demostrarle a un auditor quién movió cierto dinero hace seis meses, ¿podrías con lo que guardás hoy?

## 🔗 Relaciones
- **Anterior**: [[Fase 2 - Transferencias e Idempotencia]]
- **Siguiente**: [[Fase 4 - Hexagonal y Testing]]
- **Teoría que aplica**: [[Spring Security]], [[02 - JWT]], [[04 - Roles y Permisos]], [[12 - Auditoria y Trazabilidad]], [[07 - OWASP Top 10 en Java]]
- **Índice**: [[Proyectos Java]]

## 📌 Notas Rápidas
- El coste 12 de BCrypt es intencional: hace lento el hash y, con eso, inviable la fuerza bruta. Si el login se siente lento, funciona como debe.
- La rotación de refresh tokens con detección de reuso es un patrón concreto (*refresh token rotation*) que documenta OAuth 2.0 BCP. Vale citarlo por nombre en una entrevista.
- La auditoría inmutable no es paranoia: en banca la exige el regulador, y una tabla que admite `UPDATE` no sirve como evidencia.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

