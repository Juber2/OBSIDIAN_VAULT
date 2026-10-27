---
tags:
  - Java
  - Spring
  - Security
Creado: 2026-05-21
Relacionado:
  - Spring Security
Palabras clave:
  - SecurityFilterChain, UserDetailsService, PasswordEncoder, BCrypt, filtros
---

## 🧠 Concepto

Spring Security opera como una cadena de filtros Servlet. Cada request pasa por `SecurityFilterChain` antes de llegar al controller.

## Configuración mínima moderna (Spring Boot 3 / Spring Security 6)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()  // endpoints públicos
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()  // todo lo demás requiere auth
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)  // sin sesión (JWT)
            )
            .csrf(AbstractHttpConfigurer::disable);  // desactivar para REST APIs

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();  // estándar para hashear contraseñas
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

## UserDetailsService — cargar usuario desde BD

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UsuarioRepository usuarioRepository;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        return usuarioRepository.findByEmail(email)
            .map(usuario -> User.builder()
                .username(usuario.getEmail())
                .password(usuario.getPasswordHash())
                .roles(usuario.getRol().name())   // ROLE_USER, ROLE_ADMIN
                .accountExpired(!usuario.isActivo())
                .credentialsExpired(false)
                .accountLocked(false)
                .disabled(!usuario.isActivo())
                .build()
            )
            .orElseThrow(() -> new UsernameNotFoundException(
                "Usuario no encontrado: " + email));
    }
}
```

## Implementar UserDetails directamente en la entidad

```java
@Entity
@Table(name = "usuarios")
public class Usuario implements UserDetails {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;
    private String passwordHash;
    private boolean activo;

    @Enumerated(EnumType.STRING)
    private Rol rol;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("ROLE_" + rol.name()));
    }

    @Override
    public String getPassword() { return passwordHash; }

    @Override
    public String getUsername() { return email; }

    @Override
    public boolean isAccountNonExpired() { return true; }

    @Override
    public boolean isAccountNonLocked() { return true; }

    @Override
    public boolean isCredentialsNonExpired() { return true; }

    @Override
    public boolean isEnabled() { return activo; }
}
```

## AuthenticationProvider personalizado

```java
@Component
@RequiredArgsConstructor
public class CustomAuthenticationProvider implements AuthenticationProvider {

    private final CustomUserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;

    @Override
    public Authentication authenticate(Authentication authentication)
            throws AuthenticationException {

        String email = authentication.getName();
        String password = authentication.getCredentials().toString();

        UserDetails userDetails = userDetailsService.loadUserByUsername(email);

        if (!passwordEncoder.matches(password, userDetails.getPassword())) {
            throw new BadCredentialsException("Credenciales inválidas");
        }

        return new UsernamePasswordAuthenticationToken(
            userDetails, null, userDetails.getAuthorities());
    }

    @Override
    public boolean supports(Class<?> authenticationType) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authenticationType);
    }
}
```

## Endpoint de login con JWT (flujo completo)

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;
    private final CustomUserDetailsService userDetailsService;

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@Valid @RequestBody LoginRequest request) {
        // 1. Autenticar (lanza excepción si falla)
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.email(), request.password()));

        // 2. Cargar UserDetails
        UserDetails userDetails = userDetailsService.loadUserByUsername(request.email());

        // 3. Generar JWT
        String token = jwtService.generateToken(userDetails);

        return ResponseEntity.ok(new AuthResponse(token));
    }

    @PostMapping("/register")
    public ResponseEntity<UsuarioResponse> register(@Valid @RequestBody RegisterRequest request) {
        // lógica de registro...
    }
}

public record LoginRequest(
    @NotBlank String email,
    @NotBlank String password
) {}

public record AuthResponse(String token) {}
```

## PasswordEncoder — opciones

```java
// BCrypt (más común — recomendado)
new BCryptPasswordEncoder()           // strength por defecto: 10
new BCryptPasswordEncoder(12)         // más seguro, más lento

// Argon2 (más moderno, ganador PHC)
new Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8()

// SCrypt
new SCryptPasswordEncoder.defaultsForSpringSecurity_v5_8()

// Para tests (¡nunca en producción!)
NoOpPasswordEncoder.getInstance()

// Uso
String hash = encoder.encode("miContraseña");
boolean matches = encoder.matches("miContraseña", hash);
```

## Obtener usuario autenticado en el controller

```java
// Forma 1: @AuthenticationPrincipal
@GetMapping("/me")
public ResponseEntity<UsuarioResponse> getMe(
        @AuthenticationPrincipal UserDetails userDetails) {
    String email = userDetails.getUsername();
    // ...
}

// Forma 2: Si la entidad implementa UserDetails
@GetMapping("/me")
public ResponseEntity<UsuarioResponse> getMe(
        @AuthenticationPrincipal Usuario usuario) {
    // acceso directo a la entidad
}

// Forma 3: SecurityContextHolder (cualquier parte del código)
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String email = auth.getName();
```

## ⚠️ Errores Comunes

> [!danger] Usar field injection en SecurityConfig
> Puede causar dependencias circulares. Usar constructor injection siempre.

> [!danger] HttpSecurity vs WebSecurity
> En Spring Security 6, `WebSecurityConfigurerAdapter` está eliminado. Usar solo `SecurityFilterChain` como @Bean.

> [!warning] `hasRole` vs `hasAuthority`
> `hasRole("ADMIN")` busca `ROLE_ADMIN` — agrega el prefijo automáticamente.
> `hasAuthority("ROLE_ADMIN")` busca exacto.

## 📌 Notas Rápidas
- La cadena de filtros de Spring Security es en sí misma un Chain of Responsibility (ver [[15 - Chain of Responsibility]]) — cada filtro decide procesar o delegar al siguiente.
- BCrypt incluye el salt dentro del hash resultante — no hace falta guardar el salt por separado como en implementaciones manuales antiguas.
- `SessionCreationPolicy.STATELESS` es obligatorio en APIs con JWT — le dice a Spring que no cree ni use `HttpSession`, coherente con la naturaleza stateless del token.
- Desde Spring Security 6, todo se configura vía `SecurityFilterChain` como `@Bean` — el viejo `WebSecurityConfigurerAdapter` fue eliminado, cualquier tutorial que lo use está desactualizado.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

