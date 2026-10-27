---
tags:
  - Java
  - Testing
Creado: 2026-05-21
Relacionado:
  - Testing en Java
  - Integration Testing
Palabras clave:
  - TestContainers, Docker, PostgreSQL real, Kafka, Redis, tests reales
---

## 🧠 Concepto

**TestContainers** levanta contenedores Docker reales durante los tests. Permite testear contra bases de datos reales (PostgreSQL, MySQL, MongoDB), Kafka, Redis, etc., en vez de emuladores.

> [!important] Requiere Docker corriendo en la máquina.

## Dependencia

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.0</version>
    <scope>test</scope>
</dependency>
```

## Test con PostgreSQL real

```java
@SpringBootTest
@Testcontainers
class UsuarioRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configurarPropiedades(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UsuarioRepository repository;

    @Test
    void guardar_usuario_debePersistar() {
        Usuario usuario = new Usuario("Juber", "juber@email.com");
        Usuario guardado = repository.save(usuario);

        assertNotNull(guardado.getId());
        assertEquals("Juber", guardado.getNombre());
    }
}
```

## Reusar el contenedor entre tests (más rápido)

```java
// Clase base compartida
@SpringBootTest
@Testcontainers
public abstract class IntegrationTestBase {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withReuse(true); // reutilizar entre tests

    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}

// Test hereda la base
class UsuarioTest extends IntegrationTestBase {
    @Test void test() { ... }
}
```

## Con Kafka

```java
@Container
static KafkaContainer kafka = new KafkaContainer(
    DockerImageName.parse("confluentinc/cp-kafka:7.4.0")
);

@DynamicPropertySource
static void properties(DynamicPropertyRegistry registry) {
    registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
}
```

## Con Redis

```java
@Container
static GenericContainer<?> redis = new GenericContainer<>("redis:7")
    .withExposedPorts(6379);

@DynamicPropertySource
static void properties(DynamicPropertyRegistry registry) {
    registry.add("spring.data.redis.host", redis::getHost);
    registry.add("spring.data.redis.port", () -> redis.getMappedPort(6379));
}
```

## H2 vs TestContainers

| | H2 (in-memory) | TestContainers |
|--|---------------|----------------|
| Velocidad | Muy rápido | Más lento (Docker) |
| Fidelidad | Baja (dialecto diferente) | Alta (BD real) |
| Setup | Sin dependencias | Requiere Docker |
| Casos especiales | No soporta features específicas de BD | Soporta todo |
| Uso recomendado | Tests unitarios de repo | Tests de integración reales |

## 📌 Notas Rápidas
- `withReuse(true)` + configurar `testcontainers.reuse.enable=true` en `~/.testcontainers.properties` evita levantar un contenedor nuevo en cada ejecución local — acelera mucho el ciclo de desarrollo.
- En CI, cada pipeline suele levantar contenedores frescos (sin reuse) porque cada runner es efímero — el ahorro de reuse aplica sobre todo en local.
- Preferir TestContainers sobre H2 cuando el código usa features específicas del motor real (JSON columns, funciones nativas de Postgres) que H2 no replica fielmente.
- Es la forma estándar actual de testear integraciones con Kafka/Redis/Mongo sin depender de un servicio compartido en un ambiente de test remoto, evitando tests que "a veces fallan" por estado compartido entre corridas.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

