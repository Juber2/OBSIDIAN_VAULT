---
tags:
  - Java
  - Proyecto
  - Banca
  - corebank
  - Spring
Creado: 2026-09-08
Relacionado:
  - Proyectos Java
  - Spring Boot
  - Spring Data
Palabras clave:
  - Spring Boot, JPA, Postgres, Flyway, Bean Validation, ProblemDetail, Docker Compose
Estado:
  - Pendiente
Nivel: Intermedio
Prerequisitos: Fase 0, Spring Boot, Spring Data
---

# Fase 1 · API de cuentas

## 🎯 Objetivo

Sacar el ledger de la Fase 0 de la memoria y ponerlo detrás de una **API REST con Postgres**. Al terminar, `corebank` es un servicio que arranca con `docker compose up`, expone su contrato en Swagger y persiste de verdad.

> [!info] La regla que ordena toda la fase
> **El dominio de la Fase 0 no se toca.** `Dinero`, `Asiento` y sus invariantes se mantienen tal cual. Lo que se agrega alrededor es infraestructura: controladores, entidades y repositorios. Si terminás modificando `Dinero` para que JPA lo acepte, algo se hizo al revés.

## 🧠 Qué se aprende acá

1. **Que la entidad JPA y el objeto de dominio no son lo mismo.** El dominio modela reglas; la entidad modela filas. Mezclarlos es el atajo que después cuesta el refactor de la Fase 4.
2. **Que el contrato HTTP es público y el esquema es privado.** Exponer entidades acopla tu API al modelo de base para siempre.
3. **Que los errores también son parte del contrato.** Un 500 con stacktrace no es manejo de errores.
4. **Que el esquema se versiona.** `ddl-auto: update` funciona hasta el día que borra una columna en producción.

## 🧱 Estructura del proyecto

```
corebank/
├── docker-compose.yml
├── pom.xml
└── src/main/java/com/corebank/
    ├── dominio/                 ← viene TAL CUAL de la Fase 0
    │   ├── Dinero.java
    │   ├── Moneda.java
    │   ├── Asiento.java
    │   ├── Apunte.java
    │   └── Banco.java
    ├── persistencia/
    │   ├── entidad/             ← ClienteEntity, CuentaEntity, AsientoEntity, ApunteEntity
    │   ├── repositorio/         ← interfaces JpaRepository
    │   └── LibroJpa.java        ← implementa el Libro con base de datos
    ├── api/
    │   ├── controlador/
    │   ├── dto/                 ← records de request y response
    │   └── ManejadorDeErrores.java
    └── config/
```

## 🗄️ Esquema

```sql
-- V1__esquema_inicial.sql  (Flyway)

CREATE TABLE cliente (
    id              UUID PRIMARY KEY,
    documento       VARCHAR(20)  NOT NULL UNIQUE,
    nombre          VARCHAR(120) NOT NULL,
    email           VARCHAR(160) NOT NULL UNIQUE,
    creado_en       TIMESTAMPTZ  NOT NULL DEFAULT now()
);

CREATE TABLE cuenta (
    id              UUID PRIMARY KEY,
    numero          VARCHAR(20)  NOT NULL UNIQUE,
    cliente_id      UUID         NOT NULL REFERENCES cliente(id),
    moneda          VARCHAR(3)   NOT NULL,
    tipo            VARCHAR(20)  NOT NULL,
    estado          VARCHAR(20)  NOT NULL DEFAULT 'ACTIVA',
    creado_en       TIMESTAMPTZ  NOT NULL DEFAULT now()
);

CREATE TABLE asiento (
    id              UUID PRIMARY KEY,
    fecha           TIMESTAMPTZ  NOT NULL,
    motivo          VARCHAR(200) NOT NULL
);

CREATE TABLE apunte (
    id              BIGSERIAL PRIMARY KEY,
    asiento_id      UUID         NOT NULL REFERENCES asiento(id),
    cuenta_id       UUID         NOT NULL REFERENCES cuenta(id),
    monto           NUMERIC(19,4) NOT NULL,     -- NUNCA float ni double
    moneda          VARCHAR(3)   NOT NULL
);

CREATE INDEX idx_apunte_cuenta       ON apunte(cuenta_id);
CREATE INDEX idx_asiento_fecha       ON asiento(fecha);
CREATE INDEX idx_apunte_asiento      ON apunte(asiento_id);
```

> [!danger] `NUMERIC(19,4)`, nunca `FLOAT` ni `DOUBLE PRECISION`
> Es el mismo error de la Fase 0, ahora en la base. `NUMERIC` es exacto; los tipos de punto flotante de SQL no lo son. La escala 4 (y no 2) deja margen para cálculos intermedios de intereses y tipo de cambio.

## 💻 Piezas clave

### Contrato HTTP — records, no entidades

```java
// Request
public record CrearCuentaRequest(
        @NotNull  UUID clienteId,
        @NotNull  Moneda moneda,
        @NotNull  TipoCuenta tipo
) {}

public record MovimientoRequest(
        @NotNull @DecimalMin(value = "0.01", message = "el importe debe ser positivo")
        BigDecimal monto
) {}

// Response
public record CuentaResponse(
        UUID id,
        String numero,
        String moneda,
        String tipo,
        String estado,
        BigDecimal saldo
) {
    public static CuentaResponse de(Cuenta cuenta, Dinero saldo) {
        return new CuentaResponse(
            cuenta.id(), cuenta.numero(), cuenta.moneda().name(),
            cuenta.tipo().name(), cuenta.estado().name(), saldo.monto());
    }
}
```

### Controlador

```java
@RestController
@RequestMapping("/api/v1/cuentas")
public class CuentaController {

    private final CuentaService service;

    public CuentaController(CuentaService service) {   // inyección por constructor
        this.service = service;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public CuentaResponse crear(@Valid @RequestBody CrearCuentaRequest request) {
        return service.crear(request);
    }

    @GetMapping("/{id}")
    public CuentaResponse obtener(@PathVariable UUID id) {
        return service.obtener(id);
    }

    @PostMapping("/{id}/depositos")
    public CuentaResponse depositar(@PathVariable UUID id,
                                    @Valid @RequestBody MovimientoRequest request) {
        return service.depositar(id, request.monto());
    }

    @PostMapping("/{id}/retiros")
    public CuentaResponse retirar(@PathVariable UUID id,
                                  @Valid @RequestBody MovimientoRequest request) {
        return service.retirar(id, request.monto());
    }

    @GetMapping("/{id}/movimientos")
    public Page<MovimientoResponse> movimientos(
            @PathVariable UUID id,
            @RequestParam(required = false) Instant desde,
            @RequestParam(required = false) Instant hasta,
            Pageable pageable) {
        return service.movimientos(id, desde, hasta, pageable);
    }
}
```

> [!tip] Paginá el extracto desde el día uno
> Una cuenta con dos años de movimientos devuelve decenas de miles de filas. `Page` no es una optimización prematura: es el comportamiento correcto.

### Errores como contrato

```java
@RestControllerAdvice
public class ManejadorDeErrores {

    @ExceptionHandler(SaldoInsuficienteException.class)
    public ProblemDetail saldoInsuficiente(SaldoInsuficienteException ex) {
        var pd = ProblemDetail.forStatus(HttpStatus.UNPROCESSABLE_ENTITY);
        pd.setType(URI.create("https://corebank/errores/saldo-insuficiente"));
        pd.setTitle("Saldo insuficiente");
        pd.setDetail(ex.getMessage());
        pd.setProperty("cuenta",     ex.cuenta().valor());
        pd.setProperty("disponible", ex.disponible().monto());
        pd.setProperty("solicitado", ex.solicitado().monto());
        return pd;
    }

    @ExceptionHandler(CuentaNoEncontradaException.class)
    public ProblemDetail noEncontrada(CuentaNoEncontradaException ex) {
        var pd = ProblemDetail.forStatus(HttpStatus.NOT_FOUND);
        pd.setTitle("Cuenta no encontrada");
        pd.setDetail(ex.getMessage());
        return pd;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail validacion(MethodArgumentNotValidException ex) {
        var pd = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        pd.setTitle("Datos inválidos");
        pd.setProperty("errores", ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(FieldError::getField,
                                      FieldError::getDefaultMessage,
                                      (a, b) -> a)));
        return pd;
    }

    // Red de contención: nada de stacktraces al cliente
    @ExceptionHandler(Exception.class)
    public ProblemDetail inesperado(Exception ex) {
        log.error("error no controlado", ex);
        var pd = ProblemDetail.forStatus(HttpStatus.INTERNAL_SERVER_ERROR);
        pd.setTitle("Error interno");
        pd.setDetail("Ocurrió un error procesando la solicitud");
        return pd;
    }
}
```

`ProblemDetail` implementa **RFC 7807**, el estándar de errores HTTP. Respuesta típica:

```json
{
  "type":   "https://corebank/errores/saldo-insuficiente",
  "title":  "Saldo insuficiente",
  "status": 422,
  "detail": "La cuenta ACC-001 tiene S/ 150.00 y se solicitaron S/ 500.00",
  "cuenta": "ACC-001",
  "disponible": 150.00,
  "solicitado": 500.00
}
```

### Configuración

```yaml
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:5432/corebank
    username: ${DB_USER:corebank}
    password: ${DB_PASSWORD:}
  jpa:
    hibernate:
      ddl-auto: validate      # ← validate, NUNCA update en un proyecto serio
    open-in-view: false       # ← evita consultas lazy durante la serialización
    properties:
      hibernate.jdbc.batch_size: 50
  flyway:
    enabled: true
    locations: classpath:db/migration

springdoc:
  swagger-ui:
    path: /swagger
```

> [!warning] `open-in-view: true` es el default de Spring Boot y está mal
> Mantiene la sesión de Hibernate abierta durante la serialización de la respuesta. Eso convierte cualquier relación lazy en una consulta silenciosa a la base, en pleno render — el N+1 más difícil de detectar. Apagalo siempre.

### docker-compose.yml

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: corebank
      POSTGRES_USER: corebank
      POSTGRES_PASSWORD: local
    ports: ["5432:5432"]
    volumes: ["pgdata:/var/lib/postgresql/data"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U corebank"]
      interval: 5s
      retries: 10

  app:
    build: .
    depends_on:
      db: { condition: service_healthy }
    environment:
      DB_HOST: db
      DB_USER: corebank
      DB_PASSWORD: local
    ports: ["8080:8080"]

volumes:
  pgdata:
```

## 🌐 Endpoints de la fase

| Método | Ruta | Qué hace |
|---|---|---|
| `POST` | `/api/v1/clientes` | Alta de cliente |
| `GET` | `/api/v1/clientes/{id}` | Consulta de cliente |
| `POST` | `/api/v1/cuentas` | Apertura de cuenta |
| `GET` | `/api/v1/cuentas/{id}` | Cuenta con saldo actual |
| `GET` | `/api/v1/clientes/{id}/cuentas` | Cuentas de un cliente |
| `POST` | `/api/v1/cuentas/{id}/depositos` | Depósito |
| `POST` | `/api/v1/cuentas/{id}/retiros` | Retiro |
| `GET` | `/api/v1/cuentas/{id}/movimientos` | Extracto paginado |

Las **transferencias todavía no**: van en la Fase 2, porque necesitan idempotencia y control de concurrencia para estar bien hechas.

## ✅ Criterios de aceptación

- [ ] `docker compose up` levanta base y app, y Flyway aplica el esquema solo.
- [ ] Swagger disponible en `/swagger` con todos los endpoints documentados.
- [ ] Ninguna entidad JPA aparece en un request o response.
- [ ] Todos los errores devuelven `ProblemDetail`; ningún stacktrace llega al cliente.
- [ ] Retirar más del saldo devuelve `422` con detalle de disponible y solicitado.
- [ ] Importes inválidos (`0`, negativos, texto) devuelven `400` con el campo señalado.
- [ ] El extracto pagina y acepta rango de fechas.
- [ ] `ddl-auto: validate` y `open-in-view: false`.
- [ ] Ningún importe viaja como `double` en Java ni como `FLOAT` en SQL.
- [ ] Existe al menos un test de integración con Testcontainers que crea cuenta, deposita y consulta saldo.

## ⚠️ Trampas de esta fase

> [!warning] Las cuatro que caen siempre
> - **`@Entity` en el controlador.** Arrastra relaciones lazy, filtra el esquema y explota al serializar. Usá records de respuesta.
> - **`ddl-auto: update`.** Genera esquemas distintos en cada máquina y no deja historial. Flyway desde el primer día.
> - **Calcular el saldo con un `findAll()` y sumar en Java.** Con 50.000 movimientos te comés la memoria. Que sume la base: `SELECT COALESCE(SUM(monto),0) FROM apunte WHERE cuenta_id = ?`.
> - **Inyección por campo con `@Autowired`.** Oculta dependencias y obliga a levantar Spring para testear. Constructor siempre.

## 🔁 Retrospectiva

- ¿Cuánto código del dominio de la Fase 0 tuviste que modificar? Si fue mucho, ¿por qué?
- ¿Dónde decidiste calcular el saldo: en Java o en SQL? ¿Qué te hizo elegir?
- ¿Qué pasa hoy si dos depósitos simultáneos llegan a la misma cuenta?
- ¿Qué endpoint te resultó más difícil de diseñar y por qué?

## 🔗 Relaciones
- **Anterior**: [[Fase 0 - Ledger en Java Puro]]
- **Siguiente**: [[Fase 2 - Transferencias e Idempotencia]]
- **Teoría que aplica**: [[Spring Boot]], [[Spring Data]], [[02 - REST Controllers]], [[04 - Validaciones]], [[08 - Manejo de Errores]], [[06 - Migraciones]]
- **Índice**: [[Proyectos Java]]

## 📌 Notas Rápidas
- El `LibroJpa` implementa la misma interfaz que el `Libro` en memoria de la Fase 0. Eso permite seguir corriendo los tests de dominio sin base de datos — rápidos y sin Docker.
- La versión en la ruta (`/api/v1/`) se pone desde el principio: agregarla después rompe a todos los clientes.
- Si el saldo tarda al crecer los movimientos, **no lo materialices todavía**. Ese problema se resuelve con criterio en la Fase 8; adelantarse acá es optimización a ciegas.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

