---
tags:
  - Java
  - Spring
  - Security
Creado: 2026-05-21
Relacionado:
  - Spring Security
  - JWT
Palabras clave:
  - OAuth2, OpenID Connect, Authorization Server, Resource Server, Google, Keycloak
---

## 🧠 Conceptos

### OAuth2 — roles

| Rol | Qué es |
|-----|--------|
| **Resource Owner** | El usuario |
| **Client** | La aplicación (tu app) |
| **Authorization Server** | Quien emite tokens (Google, Keycloak, Auth0) |
| **Resource Server** | Tu API que protege recursos |

### Flujos OAuth2

```
Authorization Code (apps web — más seguro):
  User → App → Auth Server → User login → redirect con code → App intercambia code por token

Client Credentials (machine-to-machine — sin usuario):
  Service → Auth Server → access token → Resource Server

Implicit (deprecated — no usar)
Resource Owner Password (deprecated — no usar)
```

## Resource Server — proteger tu API con JWT externo

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          # Auth Server emite tokens — tu app los valida con esta URL
          issuer-uri: https://accounts.google.com
          # O especifica el JWKS endpoint directamente
          jwk-set-uri: https://www.googleapis.com/oauth2/v3/certs
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .csrf(AbstractHttpConfigurer::disable);

        return http.build();
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter =
            new JwtGrantedAuthoritiesConverter();
        // mapear claim "roles" del JWT a GrantedAuthorities
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
        grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");

        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);
        return converter;
    }
}
```

## Login con Google (OAuth2 Client)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope:
              - email
              - profile
          github:
            client-id: ${GITHUB_CLIENT_ID}
            client-secret: ${GITHUB_CLIENT_SECRET}
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .userInfoEndpoint(info -> info
                    .userService(customOAuth2UserService())
                )
            );

        return http.build();
    }
}

// Procesar el usuario de OAuth2 y guardar en BD
@Service
@RequiredArgsConstructor
public class CustomOAuth2UserService extends DefaultOAuth2UserService {

    private final UsuarioRepository usuarioRepo;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest request) throws OAuth2AuthenticationException {
        OAuth2User oauth2User = super.loadUser(request);

        String email = oauth2User.getAttribute("email");
        String nombre = oauth2User.getAttribute("name");

        // Crear o actualizar usuario en BD
        Usuario usuario = usuarioRepo.findByEmail(email)
            .orElseGet(() -> {
                Usuario nuevo = new Usuario();
                nuevo.setEmail(email);
                nuevo.setNombre(nombre);
                nuevo.setRol(Rol.USER);
                return nuevo;
            });

        usuarioRepo.save(usuario);

        return oauth2User;
    }
}
```

## Keycloak como Authorization Server

```yaml
# application.yml con Keycloak
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8180/realms/mi-realm
```

```java
// Convertir roles de Keycloak (que vienen en un claim especial)
@Bean
public JwtAuthenticationConverter jwtAuthenticationConverter() {
    JwtGrantedAuthoritiesConverter converter = new JwtGrantedAuthoritiesConverter();

    // Keycloak pone los roles en: realm_access.roles
    // Necesitas un converter personalizado:
    JwtAuthenticationConverter jwtConverter = new JwtAuthenticationConverter();
    jwtConverter.setJwtGrantedAuthoritiesConverter(jwt -> {
        Collection<GrantedAuthority> authorities = new ArrayList<>();

        // roles de realm
        Map<String, Object> realmAccess = jwt.getClaim("realm_access");
        if (realmAccess != null) {
            List<String> roles = (List<String>) realmAccess.get("roles");
            if (roles != null) {
                roles.stream()
                    .map(role -> new SimpleGrantedAuthority("ROLE_" + role.toUpperCase()))
                    .forEach(authorities::add);
            }
        }

        return authorities;
    });

    return jwtConverter;
}
```

## OpenID Connect

OpenID Connect (OIDC) es una capa de identidad sobre OAuth2. Agrega:
- **ID Token** (JWT con información del usuario)
- **UserInfo endpoint**
- **Claims estándar**: `sub`, `email`, `name`, `picture`

```
OAuth2: "Autorizas a esta app a acceder a tus datos de GitHub"
OIDC:   "Autorizas a esta app a conocer tu identidad (login con Google)"
```

## Authorization Server propio (Spring Authorization Server)

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
</dependency>
```

```java
@Configuration
public class AuthorizationServerConfig {

    @Bean
    @Order(1)
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http)
            throws Exception {
        OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
        http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
            .oidc(Customizer.withDefaults());  // Enable OpenID Connect 1.0

        return http.build();
    }

    @Bean
    public RegisteredClientRepository registeredClientRepository() {
        RegisteredClient client = RegisteredClient.withId(UUID.randomUUID().toString())
            .clientId("mi-cliente")
            .clientSecret("{bcrypt}$2a$10$...")
            .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
            .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
            .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
            .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
            .redirectUri("http://localhost:3000/callback")
            .scope("read")
            .scope("write")
            .build();

        return new InMemoryRegisteredClientRepository(client);
    }

    @Bean
    public JWKSource<SecurityContext> jwkSource() {
        RSAKey rsaKey = generateRsa();
        JWKSet jwkSet = new JWKSet(rsaKey);
        return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
    }
}
```

## 📌 Notas Rápidas
- OAuth2 es sobre **autorización** (qué puede hacer una app en tu nombre); OpenID Connect agrega **autenticación** (quién sos) encima — confundir ambos es un error común de entrevista.
- "Resource Server" (tu API validando tokens de un Auth Server externo) es distinto de implementar tu propio login — muchas apps empresariales solo necesitan ser Resource Server, no Authorization Server completo.
- Keycloak es la opción open-source más común para tener un Authorization Server propio sin depender de Google/Auth0 — común en empresas que necesitan control total de identidad.
- El flujo Client Credentials (sin usuario) es el que se usa para comunicación machine-to-machine entre microservicios, distinto del Authorization Code que es para apps con usuario humano de por medio.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

