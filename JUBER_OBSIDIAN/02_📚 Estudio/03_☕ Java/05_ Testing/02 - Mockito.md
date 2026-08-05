---
tags:
  - Java
  - Testing
Creado: 2026-05-21
Relacionado:
  - Testing en Java
  - JUnit 5
Palabras clave:
  - Mockito, mock, stub, spy, verify, when, thenReturn, ArgumentCaptor
---

## 🧠 Concepto

**Mockito** permite crear objetos simulados (mocks) de dependencias para testear unidades de código de forma aislada, sin necesitar implementaciones reales.

## Dependencia

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>5.4.0</version>
    <scope>test</scope>
</dependency>
```

## Setup con JUnit 5

```java
@ExtendWith(MockitoExtension.class)
class ServicioUsuarioTest {

    @Mock
    private UsuarioRepository repository; // mock automático

    @Mock
    private EmailService emailService;

    @InjectMocks
    private ServicioUsuario servicio; // se inyectan los mocks

    @Test
    void buscarPorId_usuarioExiste_debeRetornarUsuario() {
        // ...
    }
}
```

## Configurar comportamiento — `when / thenReturn`

```java
@Test
void buscarPorId_usuarioExiste_debeRetornarUsuario() {
    // Arrange
    Usuario usuarioEsperado = new Usuario(1L, "Juber", "juber@email.com");
    when(repository.findById(1L)).thenReturn(Optional.of(usuarioEsperado));

    // Act
    Usuario resultado = servicio.buscarPorId(1L);

    // Assert
    assertNotNull(resultado);
    assertEquals("Juber", resultado.getNombre());
}

// Retornar distintos valores en llamadas sucesivas
when(repository.findById(anyLong()))
    .thenReturn(Optional.of(usuario))
    .thenReturn(Optional.empty());

// Lanzar excepción
when(repository.findById(99L))
    .thenThrow(new RuntimeException("No encontrado"));

// Para métodos void
doThrow(new RuntimeException("Error")).when(emailService).enviar(any());
doNothing().when(emailService).enviar(any());
```

## Matchers de argumentos

```java
// Exacto
when(repo.findById(1L)).thenReturn(Optional.of(usuario));

// Cualquier valor
when(repo.findById(anyLong())).thenReturn(Optional.of(usuario));
when(repo.findByNombre(anyString())).thenReturn(List.of(usuario));

// Con condición
when(repo.findByNombre(argThat(nombre -> nombre.startsWith("J"))))
    .thenReturn(List.of(usuario));

// Null / not null
when(repo.findById(isNull())).thenThrow(new IllegalArgumentException());
when(repo.findById(notNull())).thenReturn(Optional.of(usuario));
```

## Verificar interacciones — `verify`

```java
@Test
void guardar_usuarioValido_debeGuardarYEnviarEmail() {
    // Act
    servicio.registrar(new RegistroRequest("Juber", "juber@email.com"));

    // Verify — verificar que se llamaron los métodos correctos
    verify(repository).save(any(Usuario.class));
    verify(emailService).enviarBienvenida("juber@email.com");

    // Verificar número de llamadas
    verify(repository, times(1)).save(any());
    verify(emailService, never()).enviarError(any());
    verify(repository, atLeastOnce()).save(any());
    verify(repository, atMost(2)).save(any());

    // No debe quedar ninguna interacción sin verificar
    verifyNoMoreInteractions(repository, emailService);
}
```

## ArgumentCaptor — capturar argumentos

```java
@Test
void registrar_debeGuardarConEmailCorrecto() {
    ArgumentCaptor<Usuario> captor = ArgumentCaptor.forClass(Usuario.class);

    servicio.registrar(new RegistroRequest("Juber", "JUBER@EMAIL.COM"));

    verify(repository).save(captor.capture());
    Usuario guardado = captor.getValue();

    assertEquals("juber@email.com", guardado.getEmail()); // verifica normalización
    assertNotNull(guardado.getFechaCreacion());
}
```

## Spy — mock parcial

```java
// Spy: objeto real con algunos métodos "mockeados"
@Spy
private ServicioEmail spyEmail = new ServicioEmailImpl();

// O manualmente
ServicioEmail spy = Mockito.spy(new ServicioEmailImpl());
doReturn("mock").when(spy).generarToken(); // sobreescribir solo este método
spy.enviar("destino"); // usa la implementación real
```

## ⚠️ Errores Comunes

> [!warning] `UnnecessaryStubbingException`
> Mockito avisa si configuras un `when()` que nunca se llama. Verificar que el código bajo test realmente llama ese método.

> [!warning] `when()` en métodos `void`
> Para void usar `doThrow().when()` o `doNothing().when()`, no `when().thenReturn()`.

## 📌 Notas Rápidas
