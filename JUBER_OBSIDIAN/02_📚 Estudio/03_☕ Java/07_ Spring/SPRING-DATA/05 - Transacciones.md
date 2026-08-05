---
tags:
  - Java
  - Spring
  - JPA
Creado: 2026-05-21
Relacionado:
  - Spring Data
Palabras clave:
  - @Transactional, propagación, aislamiento, rollback, ACID, transacciones
---

## 🧠 Concepto

Una **transacción** agrupa operaciones que deben ejecutarse como una unidad atómica: todas tienen éxito o todas fallan (ACID).

## @Transactional básico

```java
@Service
public class TransferenciaService {

    @Transactional  // si hay excepción → rollback automático
    public void transferir(Long origenId, Long destinoId, BigDecimal monto) {
        Cuenta origen = cuentaRepo.findById(origenId).orElseThrow();
        Cuenta destino = cuentaRepo.findById(destinoId).orElseThrow();

        if (origen.getSaldo().compareTo(monto) < 0) {
            throw new SaldoInsuficienteException("Saldo insuficiente");
        }

        origen.setSaldo(origen.getSaldo().subtract(monto));
        destino.setSaldo(destino.getSaldo().add(monto));

        cuentaRepo.save(origen);
        cuentaRepo.save(destino);
        // si cualquier línea falla → rollback de todo
    }
}
```

## Propagación (Propagation)

Define cómo se comporta la transacción si ya existe una activa.

| Propagación | Comportamiento |
|-------------|----------------|
| `REQUIRED` (default) | Usa la existente o crea una nueva |
| `REQUIRES_NEW` | Siempre crea una nueva (suspende la actual) |
| `NESTED` | Transacción anidada con savepoint |
| `SUPPORTS` | Usa la existente si hay; sin transacción si no hay |
| `NOT_SUPPORTED` | Suspende la transacción activa |
| `MANDATORY` | Requiere transacción activa, sino excepción |
| `NEVER` | Falla si hay transacción activa |

```java
@Service
public class AuditoriaService {

    // Se ejecuta en su PROPIA transacción, independiente del llamador
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void registrar(String accion, Long usuarioId) {
        // Aunque el padre falle, este commit persiste
        auditoriaRepo.save(new Auditoria(accion, usuarioId, LocalDateTime.now()));
    }
}
```

## Aislamiento (Isolation)

Controla qué cambios de otras transacciones son visibles.

| Nivel | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|---------------------|--------------|
| `READ_UNCOMMITTED` | ✅ puede | ✅ puede | ✅ puede |
| `READ_COMMITTED` (default PG) | ❌ no | ✅ puede | ✅ puede |
| `REPEATABLE_READ` | ❌ no | ❌ no | ✅ puede |
| `SERIALIZABLE` | ❌ no | ❌ no | ❌ no |

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void operacionCritica() { ... }
```

## Rollback — cuándo hace rollback

```java
// Por defecto: rollback en RuntimeException y Error
@Transactional
public void metodo() {
    throw new RuntimeException("rollback"); // ✅ hace rollback
}

// ❌ NO hace rollback con checked exceptions por defecto
@Transactional
public void metodo() throws Exception {
    throw new Exception("NO rollback"); // ❌ no rollback por defecto
}

// Forzar rollback con checked exception
@Transactional(rollbackFor = Exception.class)
public void metodo() throws Exception {
    throw new Exception("rollback"); // ✅ rollback
}

// Excluir una excepción del rollback
@Transactional(noRollbackFor = OptimisticLockingFailureException.class)
public void metodo() { ... }
```

## readOnly — optimización para lecturas

```java
@Transactional(readOnly = true) // no rastrea cambios, más eficiente
public List<Usuario> listar() {
    return repository.findAll();
}

// Patrón: service genérico con lectura optimizada
@Service
@Transactional(readOnly = true) // default en la clase
public class UsuarioService {

    public List<UsuarioResponse> listar() { ... } // hereda readOnly

    @Transactional // sobreescribe: escritura completa
    public UsuarioResponse crear(CrearUsuarioRequest req) { ... }

    @Transactional
    public void eliminar(Long id) { ... }
}
```

## ⚠️ Errores Comunes

> [!danger] @Transactional en métodos private no funciona
> Spring AOP no puede interceptar métodos privados. Moverlos a un bean público separado.

> [!danger] Llamada interna en la misma clase no pasa por el proxy
> ```java
> public void metodoA() {
>     metodoB(); // ❌ no aplica @Transactional de metodoB
> }
>
> @Transactional
> public void metodoB() { ... }
> ```
> Solución: inyectar el propio servicio o mover a otra clase.

## 📌 Notas Rápidas
