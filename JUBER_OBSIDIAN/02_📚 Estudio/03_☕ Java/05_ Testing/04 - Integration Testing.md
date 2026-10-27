---
tags:
  - Java
  - Testing
Creado: 2026-05-21
Relacionado:
  - Testing en Java
  - Spring Boot
Palabras clave:
  - integration testing, test de integración, REST Assured, MockMvc, H2
---

## 🧠 Concepto

Los **tests de integración** verifican que múltiples componentes funcionan juntos correctamente: controladores + servicios + repositorios + base de datos.

## Pirámide de testing

```
         /\
        /  \       E2E (pocos, lentos)
       /----\
      /      \     Integración (algunos, moderados)
     /--------\
    /          \   Unitarios (muchos, rápidos)
   /____________\
```

## Test de integración con Spring Boot

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.ANY) // usa H2
@Transactional // rollback después de cada test
class UsuarioControllerIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UsuarioRepository repository;

    @BeforeEach
    void setUp() {
        repository.deleteAll();
    }

    @Test
    void crearUsuario_requestValido_debeRetornar201() {
        RegistroRequest request = new RegistroRequest("Juber", "juber@email.com");

        ResponseEntity<UsuarioResponse> response = restTemplate.postForEntity(
            "/api/usuarios",
            request,
            UsuarioResponse.class
        );

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertEquals("Juber", response.getBody().getNombre());

        // Verificar en BD
        Optional<Usuario> guardado = repository.findByEmail("juber@email.com");
        assertTrue(guardado.isPresent());
    }
}
```

## MockMvc — test de capa web sin levantar servidor

```java
@WebMvcTest(UsuarioController.class) // solo carga la capa web
class UsuarioControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UsuarioService usuarioService; // mock del servicio

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void crearUsuario_requestValido_debeRetornar201() throws Exception {
        RegistroRequest request = new RegistroRequest("Juber", "juber@email.com");
        UsuarioResponse response = new UsuarioResponse(1L, "Juber", "juber@email.com");

        when(usuarioService.registrar(any())).thenReturn(response);

        mockMvc.perform(post("/api/usuarios")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.nombre").value("Juber"))
            .andExpect(jsonPath("$.email").value("juber@email.com"));
    }

    @Test
    void crearUsuario_requestInvalido_debeRetornar400() throws Exception {
        RegistroRequest request = new RegistroRequest("", "no-es-email");

        mockMvc.perform(post("/api/usuarios")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest());
    }
}
```

## Test de repositorio con `@DataJpaTest`

```java
@DataJpaTest // carga solo la capa JPA con H2 en memoria
class UsuarioRepositoryTest {

    @Autowired
    private UsuarioRepository repository;

    @Test
    void findByEmail_emailExistente_debeRetornarUsuario() {
        // Arrange
        repository.save(new Usuario("Juber", "juber@email.com"));

        // Act
        Optional<Usuario> resultado = repository.findByEmail("juber@email.com");

        // Assert
        assertTrue(resultado.isPresent());
        assertEquals("Juber", resultado.get().getNombre());
    }
}
```

## REST Assured — tests de API más expresivos

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class ApiTest {

    @LocalServerPort
    private int port;

    @BeforeEach
    void setUp() {
        RestAssured.baseURI = "http://localhost:" + port;
    }

    @Test
    void crearUsuario() {
        given()
            .contentType(ContentType.JSON)
            .body("""
                {"nombre": "Juber", "email": "juber@email.com"}
                """)
        .when()
            .post("/api/usuarios")
        .then()
            .statusCode(201)
            .body("nombre", equalTo("Juber"))
            .body("id", notNullValue());
    }
}
```

## 📌 Notas Rápidas
- `@WebMvcTest` es más rápido que `@SpringBootTest` porque no levanta toda la app (base de datos, seguridad completa, etc.) — usarlo cuando solo se necesita testear la capa de controllers.
- `@Transactional` en un test hace rollback automático al terminar — evita tener que limpiar manualmente los datos insertados en cada test.
- Seguir la pirámide de testing (muchos unitarios, algunos de integración, pocos E2E) evita suites de test lentas que desincentivan correrlas seguido.
- `MockMvc` testea la capa web sin necesitar un servidor HTTP real levantado — más rápido que `TestRestTemplate` con `RANDOM_PORT`.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

