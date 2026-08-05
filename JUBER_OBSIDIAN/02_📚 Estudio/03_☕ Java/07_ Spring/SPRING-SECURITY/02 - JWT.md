---
tags:
  - Java
  - Spring
  - Security
Creado: 2026-05-21
Relacionado:
  - Spring Security
  - Autenticación
Palabras clave:
  - JWT, JSON Web Token, JJWT, Bearer token, filtro de autenticación
---

## 🧠 ¿Qué es JWT?

JSON Web Token — estándar para transmitir información de forma segura entre partes como JSON firmado.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   ← Header (base64)
.
eyJzdWIiOiJ1c2VyQGVtYWlsLmNvbSIsInJvbCI6IlVTRVIiLCJpYXQiOjE2...  ← Payload (base64)
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature (HMAC SHA256)
```

### Partes del payload (claims)

```json
{
  "sub": "user@email.com",    // subject (quién)
  "iat": 1716300000,          // issued at
  "exp": 1716386400,          // expiration
  "rol": "USER",              // claim personalizado
  "userId": 42
}
```

## Dependencia (JJWT — librería más usada)

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.6</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.6</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.6</version>
    <scope>runtime</scope>
</dependency>
```

## Properties

```yaml
app:
  jwt:
    secret: mi-super-secreto-de-al-menos-256-bits-para-HS256-seguro
    expiration: 86400000  # 24h en milisegundos
    refresh-expiration: 604800000  # 7 días
```

## JwtService — generación y validación

```java
@Service
public class JwtService {

    @Value("${app.jwt.secret}")
    private String secretKey;

    @Value("${app.jwt.expiration}")
    private long jwtExpiration;

    // ─── Generar token ───────────────────────────────────────────────
    public String generateToken(UserDetails userDetails) {
        return generateToken(Map.of(), userDetails);
    }

    public String generateToken(Map<String, Object> extraClaims, UserDetails userDetails) {
        return Jwts.builder()
            .claims(extraClaims)
            .subject(userDetails.getUsername())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + jwtExpiration))
            .signWith(getSigningKey())
            .compact();
    }

    // ─── Validar token ────────────────────────────────────────────────
    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    // ─── Extraer claims ───────────────────────────────────────────────
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parser()
            .verifyWith(getSigningKey())
            .build()
            .parseSignedClaims(token)
            .getPayload();
    }

    private boolean isTokenExpired(String token) {
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }

    private SecretKey getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

## JwtAuthenticationFilter — interceptar requests

```java
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {

        // 1. Extraer el header Authorization
        final String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);  // sin token → continuar sin auth
            return;
        }

        // 2. Extraer el token (remover "Bearer ")
        final String jwt = authHeader.substring(7);
        final String userEmail;

        try {
            userEmail = jwtService.extractUsername(jwt);
        } catch (JwtException e) {
            // Token malformado o firma inválida
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("Token inválido");
            return;
        }

        // 3. Si hay email y no hay autenticación activa
        if (userEmail != null &&
                SecurityContextHolder.getContext().getAuthentication() == null) {

            UserDetails userDetails = userDetailsService.loadUserByUsername(userEmail);

            // 4. Validar token
            if (jwtService.isTokenValid(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authToken =
                    new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());

                authToken.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request));

                // 5. Guardar en SecurityContext
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

## Registrar el filtro en SecurityConfig

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;
    private final UserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authenticationProvider(authenticationProvider())
            // 🔑 agregar filtro JWT antes del filtro de username/password
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .csrf(AbstractHttpConfigurer::disable);

        return http.build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config)
            throws Exception {
        return config.getAuthenticationManager();
    }
}
```

## Refresh Token — patrón completo

```java
@Entity
@Table(name = "refresh_tokens")
public class RefreshToken {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String token;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "usuario_id")
    private Usuario usuario;

    private Instant expiresAt;
    private boolean revoked;
}

@Service
@RequiredArgsConstructor
public class RefreshTokenService {

    private final RefreshTokenRepository refreshTokenRepo;

    @Value("${app.jwt.refresh-expiration}")
    private long refreshExpiration;

    public RefreshToken create(Usuario usuario) {
        // revocar tokens anteriores (single session)
        refreshTokenRepo.revokeAllByUsuario(usuario);

        RefreshToken token = new RefreshToken();
        token.setToken(UUID.randomUUID().toString());
        token.setUsuario(usuario);
        token.setExpiresAt(Instant.now().plusMillis(refreshExpiration));
        token.setRevoked(false);

        return refreshTokenRepo.save(token);
    }

    public RefreshToken verify(String token) {
        RefreshToken rt = refreshTokenRepo.findByToken(token)
            .orElseThrow(() -> new RuntimeException("Refresh token no encontrado"));

        if (rt.isRevoked() || rt.getExpiresAt().isBefore(Instant.now())) {
            throw new RuntimeException("Refresh token expirado o revocado");
        }

        return rt;
    }
}
```

## Claims personalizados

```java
// Agregar claims extras al generar token
public String generateToken(UserDetails userDetails) {
    Map<String, Object> claims = new HashMap<>();

    if (userDetails instanceof Usuario usuario) {
        claims.put("userId", usuario.getId());
        claims.put("rol", usuario.getRol().name());
        claims.put("nombre", usuario.getNombre());
    }

    return generateToken(claims, userDetails);
}

// Extraer claim personalizado
public Long extractUserId(String token) {
    return extractClaim(token, claims -> claims.get("userId", Long.class));
}
```

## ⚠️ Errores Comunes

> [!danger] Guardar información sensible en el JWT
> El payload es solo Base64 — cualquiera puede decodificarlo. Nunca guardar contraseñas, datos médicos, etc. Solo IDs y roles.

> [!danger] Secret key débil
> Usar al menos 256 bits (32 caracteres). En producción: variable de entorno, nunca hardcodeado.

> [!warning] No invalidar JWT al hacer logout
> JWT es stateless — no se puede invalidar sin una blacklist (Redis). Estrategia: refresh tokens de corta duración + blacklist de JWTs revocados.

> [!tip] Expiración recomendada
> Access token: 15 min — 1 hora. Refresh token: 7 — 30 días.

## 📌 Notas Rápidas
