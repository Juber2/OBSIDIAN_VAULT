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
  - repository, DAO, persistencia, agregado, coleccion, puerto, Spring Data
---

## 🧠 Concepto

Da acceso a los objetos del dominio **como si fueran una colección en memoria**, escondiendo por completo de dónde salen: base relacional, API remota, archivo o memoria.

**Cuándo usarlo:** siempre que el dominio tenga que persistirse y quieras poder testearlo sin base de datos. Es uno de los patrones más usados y no está en el catálogo GoF — viene de *Patterns of Enterprise Application Architecture* y de DDD.

## La idea central

```java
// El dominio pide en SU lenguaje, no en el de la base
public interface CuentaRepository {
    Optional<Cuenta> porNumero(NumeroCuenta numero);
    List<Cuenta> deCliente(ClienteId cliente);
    void guardar(Cuenta cuenta);
}
```

Fijate que **no aparece nada de SQL, JPA ni tablas**. Devuelve `Cuenta` y recibe `NumeroCuenta`: tipos del dominio.

## 🆚 Repository vs DAO

Se usan como sinónimos y no lo son.

| | DAO | Repository |
|---|---|---|
| Piensa en | tablas | **agregados del dominio** |
| Granularidad | una por tabla | una por **raíz de agregado** |
| Devuelve | filas, DTOs, entidades | objetos del dominio completos |
| Lenguaje | técnico (`findByEstadoAndTipo`) | de negocio (`buscarMorosas`) |
| Origen | patrón de acceso a datos | DDD |

```java
// DAO: refleja la base
public interface CuentaDao {
    CuentaEntity selectById(UUID id);
    List<CuentaEntity> selectByClienteId(UUID clienteId);
    void insert(CuentaEntity e);
    void update(CuentaEntity e);
}

// Repository: refleja el dominio
public interface CuentaRepository {
    Optional<Cuenta> porNumero(NumeroCuenta numero);
    List<Cuenta> conSaldoNegativo();                  // una consulta de NEGOCIO
    void guardar(Cuenta cuenta);                       // no distingue insert de update
}
```

> [!tip] `guardar()` en vez de `insert()` y `update()`
> Una colección en memoria no distingue entre agregar y modificar: guardás el objeto y listo. Que por debajo sea un `INSERT` o un `UPDATE` es un detalle de la implementación, no del dominio.

## Una por agregado, no por tabla

```java
// Una transferencia toca cuenta, asiento y apunte.
// NO hay tres repositorios: hay uno por raíz de agregado.

public interface LibroRepository {
    void registrar(Asiento asiento);       // el asiento y sus apuntes son UN agregado
    Optional<Asiento> porId(UUID id);
    Dinero saldo(CuentaId cuenta, Moneda moneda);
}
```

> [!warning] Un repositorio por tabla es la señal de que estás haciendo DAO
> Si tenés `ApunteRepository` aparte de `AsientoRepository`, nada impide guardar apuntes sin su asiento — y ahí se rompe la invariante de que el asiento cuadre.
> El repositorio protege el agregado: se guarda entero o no se guarda.

## Como puerto en arquitectura hexagonal

Acá es donde el patrón realmente rinde.

```
com.corebank/
├── dominio/                    ← Cuenta, Asiento, Dinero
├── aplicacion/
│   └── puerto/salida/
│       └── CuentaRepository    ← la INTERFAZ vive acá, en el núcleo
└── infraestructura/
    └── salida/persistencia/
        └── CuentaRepositoryJpa ← la IMPLEMENTACIÓN vive afuera
```

```java
// El caso de uso depende de la interfaz, no de JPA
@Service
public class TransferirService {
    private final CuentaRepository cuentas;          // interfaz del núcleo

    public TransferirService(CuentaRepository cuentas) { this.cuentas = cuentas; }
}
```

```java
// El adaptador traduce entre dominio y persistencia
@Component
public class CuentaRepositoryJpa implements CuentaRepository {

    private final CuentaJpaRepository jpa;           // Spring Data, detalle interno
    private final CuentaMapper mapper;

    @Override
    public Optional<Cuenta> porNumero(NumeroCuenta numero) {
        return jpa.findByNumero(numero.valor()).map(mapper::aDominio);
    }

    @Override
    public void guardar(Cuenta cuenta) {
        jpa.save(mapper.aEntidad(cuenta));
    }
}
```

> [!tip] La inversión de dependencias es todo el punto
> La interfaz la define **el que la necesita** (el dominio), no el que la implementa (la infraestructura). Por eso el dominio no importa nada de JPA — y ArchUnit lo puede verificar, ver [[07 - ArchUnit]] y [[03 - Arquitectura Hexagonal]].

## Testear sin base de datos

```java
public class CuentaRepositoryEnMemoria implements CuentaRepository {

    private final Map<NumeroCuenta, Cuenta> datos = new HashMap<>();

    @Override public Optional<Cuenta> porNumero(NumeroCuenta n) {
        return Optional.ofNullable(datos.get(n));
    }
    @Override public void guardar(Cuenta c) { datos.put(c.numero(), c); }
    @Override public List<Cuenta> conSaldoNegativo() {
        return datos.values().stream().filter(c -> c.saldo().esNegativo()).toList();
    }
}
```

```java
@Test
void no_permite_transferir_sin_fondos() {
    var repo = new CuentaRepositoryEnMemoria();       // fake, no mock
    var servicio = new TransferirService(repo, ...);

    assertThrows(SaldoInsuficienteException.class, () -> servicio.transferir(cmd));
}
```

> [!tip] Un fake es mejor que un mock acá
> El fake es una implementación real y simple: verifica **comportamiento**. Un mock verifica llamadas, así que el test se rompe al refactorizar aunque el comportamiento no cambie.
> Estos tests corren en milisegundos y sin Docker.

## Con Spring Data

```java
public interface CuentaJpaRepository extends JpaRepository<CuentaEntity, UUID> {
    Optional<CuentaEntity> findByNumero(String numero);
}
```

> [!warning] Extender `JpaRepository` desde el dominio rompe el patrón
> ```java
> // ❌ El dominio ahora depende de Spring Data
> public interface CuentaRepository extends JpaRepository<Cuenta, UUID> { }
> ```
> Además `JpaRepository` expone 20 métodos que tu dominio no necesita: `deleteAll()`, `findAll()`, `flush()`. Un repositorio de dominio expone **solo lo que el negocio usa**.
>
> Para proyectos chicos, usar Spring Data directo es un atajo legítimo. Solo hay que saber que es un atajo, no el patrón.

## 🎯 Cuándo NO usarlo

| Situación | Alternativa |
|---|---|
| CRUD sin reglas de negocio | Spring Data directo |
| Consultas de reporte complejas | consulta SQL a una proyección |
| Lectura pura para una pantalla | modelo de lectura ([[09 - CQRS Distribuido]]) |

> [!danger] No metas consultas de reporte en el repositorio
> Un `CuentaRepository` con 40 métodos de búsqueda dejó de ser un repositorio: es un cajón de consultas. Las consultas de lectura que no cargan agregados van por otro camino — proyecciones o vistas.

## 📌 Notas Rápidas
- La prueba de que está bien hecho: **podés implementarlo en memoria en 20 líneas** y los tests del dominio pasan sin cambios.
- Un repositorio por **agregado**, no por tabla. Si tenés uno por cada entidad, estás haciendo DAO con otro nombre.
- El nombre de los métodos es la señal más clara: `findByEstadoAndTipoAndFechaBetween` es DAO; `buscarMorosas()` es repositorio.
- Es el patrón que hace posible el testing rápido de [[Fase 4 - Hexagonal y Testing]] y la inversión de dependencias de [[03 - Arquitectura Hexagonal]].
- No es GoF: viene de Fowler y de DDD. Aun así, es probablemente el patrón que más vas a escribir en un backend.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
