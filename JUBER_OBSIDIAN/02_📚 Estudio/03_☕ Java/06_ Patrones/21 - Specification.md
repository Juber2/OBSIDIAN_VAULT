---
tags:
  - Java
  - Patrones
  - No-GoF
  - Arquitectura
Creado: 2026-09-10
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - specification, criterio, reglas de negocio, componible, and, or, predicado
---

## 🧠 Concepto

Encapsula una **regla de negocio** en un objeto que puede combinarse con otros usando `and`, `or` y `not`.

**Cuándo usarlo:** cuando las mismas reglas se usan en varios lugares —validar, filtrar en memoria, consultar la base— y se combinan de formas distintas. Viene de DDD, no del catálogo GoF.

## El problema

```java
// La misma regla, escrita tres veces de tres formas distintas
public boolean puedeOperarcuenta(Cuenta c) {                    // validación
    return c.estado() == ACTIVA && !c.tieneEmbargo() && c.saldo().esPositivo();
}

var operables = cuentas.stream()                                 // filtro en memoria
    .filter(c -> c.estado() == ACTIVA && !c.tieneEmbargo() && c.saldo().signum() > 0)
    .toList();

@Query("SELECT c FROM Cuenta c WHERE c.estado = 'ACTIVA' " +     // consulta SQL
       "AND c.embargo = false AND c.saldo > 0")
List<Cuenta> buscarOperables();
```

Tres copias de la misma regla. Cuando el negocio agregue una condición, hay que acordarse de las tres.

## Implementación

```java
public interface Especificacion<T> {

    boolean cumple(T candidato);

    default Especificacion<T> y(Especificacion<T> otra) {
        return c -> this.cumple(c) && otra.cumple(c);
    }

    default Especificacion<T> o(Especificacion<T> otra) {
        return c -> this.cumple(c) || otra.cumple(c);
    }

    default Especificacion<T> no() {
        return c -> !this.cumple(c);
    }
}
```

```java
public class CuentaActiva implements Especificacion<Cuenta> {
    @Override public boolean cumple(Cuenta c) { return c.estado() == ACTIVA; }
}

public class SinEmbargo implements Especificacion<Cuenta> {
    @Override public boolean cumple(Cuenta c) { return !c.tieneEmbargo(); }
}

public class SaldoMayorA implements Especificacion<Cuenta> {
    private final Dinero minimo;
    public SaldoMayorA(Dinero minimo) { this.minimo = minimo; }
    @Override public boolean cumple(Cuenta c) { return c.saldo().mayorQue(minimo); }
}
```

```java
// La regla de negocio, escrita UNA vez y con nombre
public static final Especificacion<Cuenta> OPERABLE =
    new CuentaActiva().y(new SinEmbargo()).y(new SaldoMayorA(Dinero.cero(PEN)));
```

```java
// Se usa igual en los tres lugares
if (!OPERABLE.cumple(cuenta)) throw new CuentaNoOperableException();

var operables = cuentas.stream().filter(OPERABLE::cumple).toList();
```

> [!tip] La ganancia real es que la regla tiene nombre
> `OPERABLE` se lee y se busca. `estado == ACTIVA && !embargo && saldo > 0` hay que interpretarlo cada vez, y cuando aparece en cuatro archivos nadie sabe si son la misma regla o cuatro parecidas.

## Combinaciones que el negocio pide

```java
Especificacion<Cuenta> paraCobroAutomatico =
    OPERABLE.y(new TieneDebitoAutomatico()).y(new SaldoMayorA(montoDelDebito));

Especificacion<Cuenta> requiereRevision =
    new CuentaActiva().y(new SaldoMayorA(UMBRAL_ALTO).o(new MovimientosInusuales()));

Especificacion<Cuenta> paraCierreAutomatico =
    new SinMovimientosDesde(Period.ofYears(2)).y(new SaldoCero()).y(new SinEmbargo().no());
```

Cada combinación es una línea que se lee como el enunciado del negocio.

## 🗄️ Llevarlo a SQL: Spring Data Specifications

Filtrar en memoria no sirve con un millón de cuentas: hay que traducir la regla a SQL. Spring Data trae su propia versión del patrón.

```java
public interface CuentaJpaRepository
        extends JpaRepository<CuentaEntity, UUID>, JpaSpecificationExecutor<CuentaEntity> { }
```

```java
public class EspecificacionesCuenta {

    public static Specification<CuentaEntity> activa() {
        return (raiz, consulta, cb) -> cb.equal(raiz.get("estado"), "ACTIVA");
    }

    public static Specification<CuentaEntity> sinEmbargo() {
        return (raiz, consulta, cb) -> cb.isFalse(raiz.get("embargo"));
    }

    public static Specification<CuentaEntity> saldoMayorA(BigDecimal minimo) {
        return (raiz, consulta, cb) -> cb.greaterThan(raiz.get("saldo"), minimo);
    }

    public static Specification<CuentaEntity> deCliente(UUID clienteId) {
        return (raiz, consulta, cb) -> cb.equal(raiz.get("clienteId"), clienteId);
    }
}
```

```java
import static org.springframework.data.jpa.domain.Specification.*;

var spec = where(activa()).and(sinEmbargo()).and(saldoMayorA(BigDecimal.ZERO));

Page<CuentaEntity> resultado = repo.findAll(spec, pageable);
// Genera UNA consulta SQL con todos los WHERE combinados
```

> [!tip] Resuelve el problema de la búsqueda con filtros opcionales
> Es el caso donde más rinde: una pantalla con seis filtros que el usuario puede combinar como quiera.
> ```java
> Specification<CuentaEntity> spec = where(null);
> if (filtro.estado() != null)   spec = spec.and(conEstado(filtro.estado()));
> if (filtro.clienteId() != null) spec = spec.and(deCliente(filtro.clienteId()));
> if (filtro.saldoMin() != null) spec = spec.and(saldoMayorA(filtro.saldoMin()));
> return repo.findAll(spec, pageable);
> ```
> La alternativa es un método por combinación (`findByEstado`, `findByEstadoAndCliente`, `findByEstadoAndClienteAndSaldo`…) — que crece exponencialmente.

## 🆚 Specification vs Criteria vs Query Methods

| | Query methods | `@Query` | Specification |
|---|---|---|---|
| Filtros fijos | ✅ ideal | ✅ | innecesario |
| Filtros opcionales combinables | ❌ explosión de métodos | ❌ SQL con `IS NULL OR` | ✅ |
| Reglas reutilizables | ❌ | ❌ | ✅ |
| Legible | ✅ | ✅ | ⚠️ la Criteria API es verbosa |
| Verificación en compilación | ❌ nombres por convención | ❌ | parcial |

> [!warning] La Criteria API de JPA es notoriamente verbosa
> `(raiz, consulta, cb) -> cb.equal(raiz.get("estado"), "ACTIVA")` es mucho ruido para `estado = 'ACTIVA'`, y `raiz.get("estado")` es un **string**: si renombrás el campo, falla en ejecución, no al compilar.
> Alternativas: el metamodelo de JPA (`CuentaEntity_.estado`, generado en compilación) o QueryDSL, que es bastante más legible.

## ⚠️ El riesgo de las dos implementaciones

```java
// En el dominio
public static final Especificacion<Cuenta> OPERABLE = ...;

// En persistencia
public static Specification<CuentaEntity> operable() { ... }
```

> [!danger] Dos definiciones de la misma regla es exactamente lo que el patrón quería evitar
> Si el negocio agrega una condición y solo actualizás una, el sistema valida distinto de lo que consulta. Es un bug silencioso y difícil de detectar.
>
> Salidas posibles:
> - **Un solo lugar**: definir la regla en el dominio y traducirla con un visitante a SQL (ver [[19 - Visitor]]).
> - **Aceptar la duplicación** conscientemente, con un test que verifique que las dos coinciden:
> ```java
> @Test
> void la_especificacion_de_dominio_y_la_de_SQL_dan_el_mismo_resultado() {
>     var enMemoria = todas.stream().filter(OPERABLE::cumple).toList();
>     var enBase = repo.findAll(operable());
>     assertEquals(idsDe(enMemoria), idsDe(enBase));
> }
> ```

## 🎯 Cuándo NO usarlo

| Situación | Alternativa |
|---|---|
| Una sola regla, usada en un solo lugar | un `if` |
| Filtros fijos y conocidos | query method |
| El equipo no lo conoce | la indirección confunde más de lo que ayuda |

## 📌 Notas Rápidas
- La señal de que lo necesitás: **la misma condición aparece en tres lugares** y se combina distinto en cada uno.
- La mayor ganancia no es técnica sino de lenguaje: `OPERABLE.y(TIENE_DEBITO_AUTOMATICO)` se lee como el enunciado del negocio, y ese nombre es lo que después usás para hablar con el área funcional.
- Con Spring Data, `JpaSpecificationExecutor` es la herramienta estándar para búsquedas con filtros opcionales. Es lo que evita los treinta métodos `findByAAndBAndC`.
- Cuidado con mantener dos versiones de la misma regla (dominio y SQL): o unificás, o ponés un test que las compare.
- Se relaciona con [[20 - Repository]] —donde vive la consulta— y con [[11 - Strategy]], del que es primo: ambos encapsulan una decisión en un objeto.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
