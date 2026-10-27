---
tags:
  - Java
  - Lombok
Creado: 2026-08-12
Relacionado:
  - Spring Core
---

# Lombok

Librería que genera código boilerplate (getters, setters, constructores, `equals`/`hashCode`, logging) en **tiempo de compilación** vía anotaciones, mediante un procesador de anotaciones que manipula el AST del compilador — no es Reflection en runtime.

## Dependencia

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

`optional=true` porque Lombok solo se necesita en compile-time, no debería propagarse como dependencia transitiva a quien consuma tu librería.

## Anotaciones más usadas

```java
@Getter
@Setter
public class Producto {
    private Long id;
    private String nombre;
    private BigDecimal precio;
}
```

```java
@Data   // combina @Getter + @Setter + @ToString + @EqualsAndHashCode + @RequiredArgsConstructor
public class ProductoDTO {
    private Long id;
    private String nombre;
}
```

```java
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor   // constructor solo con campos `final` o `@NonNull`
public class Usuario {
    private final Long id;
    private String nombre;
}
```

```java
@Builder
public class Pedido {
    private Long id;
    private BigDecimal total;
}
// Pedido.builder().id(1L).total(BigDecimal.TEN).build();
```

```java
@Slf4j   // genera: private static final Logger log = LoggerFactory.getLogger(Clase.class);
@Service
public class PedidoService {
    public void procesar() {
        log.info("Procesando pedido");
    }
}
```

## Constructor injection sin boilerplate (el uso más común en Spring)

```java
@Service
@RequiredArgsConstructor   // genera constructor con todos los campos `final`
public class PedidoService {
    private final PedidoRepository pedidoRepository;
    private final NotificacionService notificacionService;
    // sin @Autowired, sin constructor escrito a mano
}
```

Esta combinación (`@RequiredArgsConstructor` + campos `final`) es el patrón estándar de facto para inyección de dependencias en Spring moderno — evita el anti-patrón de `@Autowired` en campos.

## @Value — versión inmutable de @Data

```java
@Value   // todos los campos son final, sin setters, clase final
public class Coordenada {
    double lat;
    double lng;
}
```

Ideal para value objects — no confundir con `@Value("${propiedad}")` de Spring (inyección de properties), es una anotación distinta con el mismo nombre.

## equals/hashCode con cuidado en entidades JPA

```java
@Entity
@Getter
@Setter
@EqualsAndHashCode(of = "id")   // ⚠️ solo por id, no por todos los campos
public class Usuario {
    @Id
    @GeneratedValue
    private Long id;
    private String nombre;
}
```

Generar `equals`/`hashCode` con **todos** los campos en una entidad JPA es un error común: antes de persistir, el `id` es `null`, y si además hay relaciones bidireccionales, `@Data` puede generar un `StackOverflowError` (toString/equals recursivo entre entidades relacionadas).

## Cómo funciona (para entrevista)

Lombok se ejecuta como un **annotation processor** durante la compilación: lee el AST (árbol de sintaxis) generado por el compilador, lo modifica agregando los métodos generados, y produce el bytecode final ya con ese código incluido. Por eso el código fuente no muestra los getters/setters, pero el `.class` compilado sí los tiene — a diferencia de Reflection, no hay costo en runtime.

## Errores comunes

- Usar `@Data` en entidades JPA con relaciones — genera `equals`/`hashCode`/`toString` sobre todos los campos, incluidas colecciones relacionadas, causando bucles infinitos o problemas de performance al cargar relaciones lazy accidentalmente.
- No configurar el IDE con el plugin de Lombok — sin él, el editor marca error en código que en realidad compila bien (Lombok genera el código que el IDE no "ve" sin el plugin).
- Mezclar `@Autowired` en campos con `@RequiredArgsConstructor` — son dos formas de inyección redundantes, elegir una (constructor injection es la recomendada).
- Usar `@Data` en vez de `@Getter`/`@Setter` explícitos cuando no se necesita `equals`/`hashCode`/`toString` generados — más código generado del necesario, puede ocultar bugs sutiles.

## Checklist
- [ ] Sé usar `@Getter`/`@Setter`/`@RequiredArgsConstructor` en vez de escribir boilerplate a mano.
- [ ] Entiendo por qué `@RequiredArgsConstructor` + campos `final` es el patrón recomendado para DI en Spring.
- [ ] Sé por qué `@Data` es riesgoso en entidades JPA con relaciones, y uso `@EqualsAndHashCode(of = "id")` en su lugar.
- [ ] Puedo explicar que Lombok actúa en compile-time (annotation processing), no en runtime como Reflection.
- [ ] Tengo el plugin de Lombok instalado en mi IDE para que no muestre falsos errores.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

