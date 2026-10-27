---
tags:
  - Java
  - Spring
  - Security
Creado: 2026-05-21
Relacionado:
  - Spring Security
Palabras clave:
  - @PreAuthorize, @Secured, hasRole, hasAuthority, Method Security, RBAC, ABAC
---

## 🧠 Roles vs Authorities

| Concepto | En Spring Security | Convención |
|----------|-------------------|------------|
| **Role** | `ROLE_ADMIN`, `ROLE_USER` | Prefijo `ROLE_` automático en `hasRole()` |
| **Authority** | `read:users`, `create:post` | String libre |

```java
// Equivalentes:
hasRole("ADMIN")         // busca ROLE_ADMIN
hasAuthority("ROLE_ADMIN")  // exacto

// Granular con authorities:
hasAuthority("users:read")
hasAuthority("users:write")
hasAuthority("orders:delete")
```

## Habilitar Method Security

```java
@Configuration
@EnableMethodSecurity  // reemplaza @EnableGlobalMethodSecurity en Spring 6
public class SecurityConfig { ... }
```

## @PreAuthorize — la más usada

```java
@RestController
@RequestMapping("/api/usuarios")
public class UsuarioController {

    // Solo ADMIN puede listar usuarios
    @GetMapping
    @PreAuthorize("hasRole('ADMIN')")
    public List<UsuarioResponse> listar() { ... }

    // ADMIN o el propio usuario puede ver su perfil
    @GetMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
    public UsuarioResponse getById(@PathVariable Long id) { ... }

    // Authority granular
    @DeleteMapping("/{id}")
    @PreAuthorize("hasAuthority('users:delete')")
    public void eliminar(@PathVariable Long id) { ... }

    // Lógica más compleja con SpEL
    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN') or " +
                  "(hasRole('USER') and #id == authentication.principal.id)")
    public UsuarioResponse actualizar(@PathVariable Long id, @RequestBody UpdateRequest req) {
        ...
    }
}
```

## @PostAuthorize — verificar después de ejecutar

```java
// Útil cuando necesitas verificar el resultado
@GetMapping("/{id}")
@PostAuthorize("returnObject.email == authentication.principal.username or hasRole('ADMIN')")
public UsuarioResponse getById(@PathVariable Long id) {
    return usuarioService.findById(id);  // se ejecuta primero, luego verifica
}
```

## @PreFilter / @PostFilter — filtrar colecciones

```java
// Filtrar lista de entrada (solo procesar elementos propios)
@PreFilter("filterObject.usuarioId == authentication.principal.id")
public void procesarPedidos(List<Pedido> pedidos) { ... }

// Filtrar lista de salida (solo devolver elementos propios)
@PostFilter("filterObject.usuarioId == authentication.principal.id or hasRole('ADMIN')")
public List<Pedido> getMisPedidos() {
    return pedidoRepository.findAll();  // filtrado antes de devolver
}
```

## @Secured — alternativa simple

```java
@Secured("ROLE_ADMIN")  // solo un rol
public void operacionAdmin() { ... }

@Secured({"ROLE_ADMIN", "ROLE_MODERATOR"})  // múltiples roles (OR)
public void operacionMod() { ... }
```

> [!tip] Preferir @PreAuthorize
> Más expresivo, soporta SpEL, roles y authorities. `@Secured` solo acepta roles.

## Autorización en SecurityFilterChain (por URL)

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth
        // Específico primero, general después
        .requestMatchers("/api/auth/**").permitAll()
        .requestMatchers(HttpMethod.GET, "/api/productos/**").permitAll()
        .requestMatchers("/api/admin/**").hasRole("ADMIN")
        .requestMatchers("/api/moderacion/**").hasAnyRole("ADMIN", "MODERADOR")
        .requestMatchers(HttpMethod.DELETE, "/api/**").hasRole("ADMIN")
        .requestMatchers("/actuator/**").hasAuthority("monitoring:read")
        .anyRequest().authenticated()
    );

    return http.build();
}
```

## Permisos basados en dominio (Domain Object Security)

```java
// Definir qué puede hacer un usuario sobre un objeto específico
@Service
public class PedidoPermissionEvaluator implements PermissionEvaluator {

    @Override
    public boolean hasPermission(Authentication auth, Object targetDomainObject,
                                  Object permission) {
        if (targetDomainObject instanceof Pedido pedido) {
            String username = auth.getName();
            return switch (permission.toString()) {
                case "READ" -> pedido.getUsuario().getEmail().equals(username)
                               || hasRole(auth, "ADMIN");
                case "DELETE" -> hasRole(auth, "ADMIN");
                default -> false;
            };
        }
        return false;
    }

    @Override
    public boolean hasPermission(Authentication auth, Serializable targetId,
                                  String targetType, Object permission) {
        // para cuando solo tienes el ID, no el objeto
        return false;
    }

    private boolean hasRole(Authentication auth, String role) {
        return auth.getAuthorities().stream()
            .anyMatch(a -> a.getAuthority().equals("ROLE_" + role));
    }
}

// Registrar el evaluador
@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig {
    @Bean
    public MethodSecurityExpressionHandler methodSecurityExpressionHandler(
            PedidoPermissionEvaluator permissionEvaluator) {
        DefaultMethodSecurityExpressionHandler handler =
            new DefaultMethodSecurityExpressionHandler();
        handler.setPermissionEvaluator(permissionEvaluator);
        return handler;
    }
}

// Usar en controller
@GetMapping("/{id}")
@PreAuthorize("hasPermission(#id, 'Pedido', 'READ')")
public PedidoResponse getPedido(@PathVariable Long id) { ... }
```

## Modelo RBAC completo — entidades

```java
@Entity
public class Usuario {
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "usuario_roles")
    private Set<Rol> roles = new HashSet<>();

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles.stream()
            .flatMap(rol -> rol.getPermisos().stream())
            .map(permiso -> new SimpleGrantedAuthority(permiso.getNombre()))
            .collect(Collectors.toSet());
    }
}

@Entity
public class Rol {
    private String nombre;  // "ADMIN", "USER", "MODERADOR"

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "rol_permisos")
    private Set<Permiso> permisos;
}

@Entity
public class Permiso {
    private String nombre;  // "users:read", "users:write", "orders:delete"
}
```

## SecurityContextHolder — acceso programático

```java
// Obtener usuario actual desde cualquier capa
public static Optional<String> getCurrentUsername() {
    return Optional.ofNullable(SecurityContextHolder.getContext().getAuthentication())
        .filter(auth -> auth.isAuthenticated())
        .map(Authentication::getName);
}

public static boolean hasRole(String role) {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    if (auth == null) return false;
    return auth.getAuthorities().stream()
        .anyMatch(a -> a.getAuthority().equals("ROLE_" + role));
}
```

## 📌 Notas Rápidas
- `@PreAuthorize` es en el fondo AOP (ver [[04 - AOP]]) — Spring intercepta la llamada al método antes de ejecutarlo, igual mecanismo que `@Transactional`.
- Roles (`ROLE_ADMIN`) son para categorías amplias de usuario; Authorities granulares (`users:delete`) dan control fino tipo RBAC/permission-based — un sistema real suele combinar ambos.
- `@PostAuthorize` es más costoso que `@PreAuthorize` porque el método ya se ejecutó antes de verificar el permiso — usarlo solo cuando la decisión de autorización depende del resultado (ej. "es dueño del recurso devuelto").
- Autorización a nivel de método (`@PreAuthorize`) y a nivel de URL (`SecurityFilterChain`) son complementarias, no excluyentes — la de URL es la primera barrera, la de método da control más fino sobre casos específicos (ej. "solo el dueño del recurso").

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

