---
tags:
  - Java
  - Quarkus
  - Testing
Creado: 2026-08-09
Relacionado:
  - Quarkus
---

# Testing en Quarkus

Quarkus se integra con JUnit 5 y trae anotaciones propias para levantar el contexto de la app en tests — equivalente a `@SpringBootTest`.

## Test de integración básico

```java
@QuarkusTest
class ProductoResourceTest {

    @Test
    void deberiaListarProductos() {
        given()
            .when().get("/api/productos")
            .then()
            .statusCode(200)
            .body("size()", greaterThanOrEqualTo(0));
    }

    @Test
    void deberiaCrearProducto() {
        var dto = new ProductoDTO(null, "Mouse", new BigDecimal("25.00"));

        given()
            .contentType(ContentType.JSON)
            .body(dto)
            .when().post("/api/productos")
            .then()
            .statusCode(201)
            .body("nombre", is("Mouse"));
    }
}
```

`@QuarkusTest` levanta la app completa (como `@SpringBootTest`) y expone **REST-assured** para probar los endpoints directamente por HTTP.

## Test unitario (sin levantar todo el contexto)

Para lógica pura sin dependencias del framework, no hace falta `@QuarkusTest` — un test JUnit normal alcanza:

```java
class CalculadoraDescuentoTest {

    @Test
    void aplicaDescuentoCorrectamente() {
        var resultado = CalculadoraDescuento.aplicar(new BigDecimal("100"), 10);
        assertEquals(new BigDecimal("90.00"), resultado);
    }
}
```

## Inyectar beans en tests

```java
@QuarkusTest
class ProductoServiceTest {

    @Inject
    ProductoService productoService;

    @Test
    void deberiaCalcularTotal() {
        var total = productoService.calcularTotal(List.of(1L, 2L));
        assertNotNull(total);
    }
}
```

## Mocks con @InjectMock (Quarkus + Mockito)

```java
@QuarkusTest
class PedidoServiceTest {

    @InjectMock
    NotificacionService notificacionService;

    @Inject
    PedidoService pedidoService;

    @Test
    void deberiaNotificarAlCrearPedido() {
        pedidoService.crear(new Pedido(/* ... */));
        verify(notificacionService).enviar(anyString());
    }
}
```

Requiere la extension `quarkus-junit5-mockito`.

## Base de datos para tests: Testcontainers

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-test-common</artifactId>
    <scope>test</scope>
</dependency>
```

Con la extension de datasource + Testcontainers, Quarkus puede levantar automáticamente un contenedor de Postgres real para los tests de integración (Dev Services) sin configurar nada manualmente — al correr `quarkus:dev` o los tests, detecta que no hay datasource configurado y levanta un container Postgres descartable solo.

## Native tests (validar que la imagen nativa también funciona)

```java
@QuarkusIntegrationTest
class ProductoResourceIT extends ProductoResourceTest {
    // hereda los mismos tests, pero corre contra el binario nativo compilado
}
```

Importante: algunos bugs (reflection no declarada, etc.) **solo aparecen en el binario nativo**, no en JVM mode — por eso conviene correr al menos los tests críticos también como `@QuarkusIntegrationTest` antes de un release.

## Errores comunes

- Usar `@QuarkusTest` para todo, incluso lógica pura sin dependencias — más lento de lo necesario, un test JUnit plano alcanza.
- No validar nunca contra la imagen nativa y descubrir un bug de reflection recién en producción.
- Depender de una base de datos real compartida en CI en vez de Dev Services/Testcontainers, generando tests frágiles.

## Checklist
- [ ] Sé escribir tests de integración con `@QuarkusTest` + REST-assured.
- [ ] Distingo cuándo un test unitario simple alcanza vs cuándo necesito el contexto completo.
- [ ] Sé mockear beans con `@InjectMock`.
- [ ] Uso Dev Services/Testcontainers para no depender de infraestructura externa en tests.
- [ ] Corro al menos los tests críticos contra el binario nativo (`@QuarkusIntegrationTest`) antes de un release.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

