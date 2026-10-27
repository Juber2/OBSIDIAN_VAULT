---
tags:
  - Java
  - Testing
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - Testing en Java
  - Spring Boot
Palabras clave:
  - @SpringBootTest, @WebMvcTest, @DataJpaTest, @MockBean, slices
---

## 🧠 Slices de test en Spring Boot

Spring Boot provee "slices" — cargan solo la parte del contexto que necesitas, haciendo los tests más rápidos.

| Anotación | Qué carga | Cuándo usar |
|-----------|-----------|-------------|
| `@SpringBootTest` | Contexto completo | Tests E2E, integración total |
| `@WebMvcTest` | Solo capa web (Controllers) | Testear endpoints |
| `@DataJpaTest` | Solo JPA + BD in-memory | Testear repositorios |
| `@DataMongoTest` | Solo MongoDB | Testear repos Mongo |
| `@RestClientTest` | RestTemplate/WebClient | Testear clientes HTTP |
| `@JsonTest` | Jackson serialización | Testear JSON |

## @SpringBootTest

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class FullIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UsuarioRepository repository;

    @Test
    void flujoCompleto() {
        // Test E2E completo
    }
}
```

## @WebMvcTest

```java
@WebMvcTest(UsuarioController.class)
class UsuarioControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean  // Reemplaza el bean real con un mock en el contexto Spring
    private UsuarioService usuarioService;

    @Test
    void getUsuario_idExistente_debeRetornar200() throws Exception {
        when(usuarioService.buscarPorId(1L))
            .thenReturn(new UsuarioResponse(1L, "Juber", "juber@email.com"));

        mockMvc.perform(get("/api/usuarios/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.nombre").value("Juber"))
            .andDo(print()); // imprime request/response en consola
    }

    @Test
    void getUsuario_idNoExistente_debeRetornar404() throws Exception {
        when(usuarioService.buscarPorId(99L))
            .thenThrow(new UsuarioNotFoundException(99L));

        mockMvc.perform(get("/api/usuarios/99"))
            .andExpect(status().isNotFound());
    }
}
```

## @DataJpaTest

```java
@DataJpaTest
class PedidoRepositoryTest {

    @Autowired
    private TestEntityManager entityManager; // para insertar datos de prueba

    @Autowired
    private PedidoRepository repository;

    @Test
    void findByUsuarioId_debeRetornarPedidosDelUsuario() {
        // Arrange — insertar datos de prueba
        Usuario usuario = entityManager.persist(new Usuario("Juber", "juber@email.com"));
        entityManager.persist(new Pedido(usuario, 100.0));
        entityManager.persist(new Pedido(usuario, 200.0));
        entityManager.flush();

        // Act
        List<Pedido> pedidos = repository.findByUsuarioId(usuario.getId());

        // Assert
        assertEquals(2, pedidos.size());
    }
}
```

## Testear con JWT

```java
@WebMvcTest(UsuarioController.class)
class UsuarioControllerSecuredTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private JwtService jwtService;

    @Test
    @WithMockUser(roles = "ADMIN") // simular usuario autenticado
    void eliminarUsuario_conRolAdmin_debeRetornar200() throws Exception {
        mockMvc.perform(delete("/api/usuarios/1"))
            .andExpect(status().isOk());
    }

    @Test
    void eliminarUsuario_sinAutenticar_debeRetornar401() throws Exception {
        mockMvc.perform(delete("/api/usuarios/1"))
            .andExpect(status().isUnauthorized());
    }
}
```

## application-test.properties

```properties
# src/test/resources/application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.hibernate.ddl-auto=create-drop
logging.level.org.springframework=WARN
```

## 📌 Notas Rápidas
- Elegir el slice más chico posible (`@WebMvcTest`, `@DataJpaTest`) en vez de `@SpringBootTest` por defecto — reduce drásticamente el tiempo de la suite completa a medida que el proyecto crece.
- `@MockBean` reemplaza un bean real en el contexto de Spring por un mock — distinto de `@Mock` de Mockito puro, que no interactúa con el contexto de Spring en absoluto.
- `@WithMockUser` simula un usuario autenticado sin necesitar generar un JWT real — suficiente para testear autorización a nivel de roles en la capa web.
- Un `application-test.properties` separado (con H2 o config mínima) evita que los tests dependan accidentalmente de configuración de dev/prod.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

