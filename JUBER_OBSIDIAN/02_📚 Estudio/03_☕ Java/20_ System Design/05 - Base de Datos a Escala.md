---
tags:
  - Java
  - System Design
  - Datos
Creado: 2026-09-08
Relacionado:
  - System Design
  - 03 - Indices y Queries Lentas
Palabras clave:
  - indices, replicas, particionado, desnormalizacion, vista materializada, N+1, EXPLAIN
---

## 🧠 Concepto

La base de datos es casi siempre el primer cuello de botella de un sistema, y también donde más se gana optimizando. La buena noticia es que **hay un orden de intervención** y el sharding está al final, no al principio.

```
1 · Índices y consultas          ← 10-1000× de mejora, costo bajo
2 · Caché                        ← quita el 80-95% de las lecturas
3 · Réplicas de lectura          ← escala lecturas linealmente
4 · Instancia más grande         ← 2-10×, sin complejidad
5 · Desnormalización y vistas    ← consultas caras que se repiten
6 · Particionado local           ← tablas enormes
7 · Sharding                     ← último recurso ([[12 - Particionado y Sharding]])
```

> [!danger] El error de arquitectura más caro es saltar al paso 7
> Shardear rompe transacciones, `JOIN` y unicidad global. Es la decisión más difícil de revertir. Antes de considerarla, **agotá los seis pasos anteriores con métricas que lo demuestren**.

## 🔍 Paso 1 — Encontrar el problema

```sql
-- Las consultas que más tiempo TOTAL consumen (no las más lentas)
SELECT calls,
       round(mean_exec_time::numeric, 2) AS media_ms,
       round(total_exec_time::numeric, 2) AS total_ms,
       query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```

> [!tip] Optimizá por tiempo total, no por tiempo medio
> Una consulta de 2 segundos que corre 10 veces por día cuesta 20 segundos. Una de 50 ms que corre un millón de veces cuesta 14 horas. **La segunda es el problema**, aunque la primera se vea peor en un listado de consultas lentas.

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM apunte WHERE cuenta_id = '...' ORDER BY id DESC LIMIT 50;
```

| Qué buscar | Qué significa |
|---|---|
| `Seq Scan` en tabla grande | falta un índice |
| `Rows Removed by Filter` alto | el índice no es selectivo |
| Diferencia entre filas estimadas y reales | estadísticas desactualizadas (`ANALYZE`) |
| `Sort` con disco | `work_mem` insuficiente |
| `Nested Loop` con muchas iteraciones | falta índice en el lado interno |

## 📇 Índices

```sql
-- Compuesto: el ORDEN de las columnas importa
CREATE INDEX idx_apunte_cuenta_fecha ON apunte(cuenta_id, fecha DESC);
```

> [!warning] La regla del prefijo izquierdo
> Un índice `(a, b, c)` sirve para consultas por `a`, por `(a, b)` y por `(a, b, c)`. **No sirve** para consultas solo por `b` o solo por `c`.
> El orden correcto: primero las columnas de igualdad, después las de rango u ordenamiento.

```sql
-- Índice cubriente: la consulta se resuelve sin tocar la tabla
CREATE INDEX idx_apunte_cubriente
    ON apunte(cuenta_id, fecha DESC) INCLUDE (monto, moneda);

-- Índice parcial: solo lo que se consulta, mucho más chico
CREATE INDEX idx_outbox_pendientes ON outbox(creado_en)
    WHERE publicado_en IS NULL;

-- Índice de expresión
CREATE INDEX idx_cliente_email_lower ON cliente(lower(email));
```

> [!tip] Los índices parciales son de lo más rentable que hay
> La tabla `outbox` acumula millones de filas publicadas y solo decenas pendientes. Un índice sobre toda la tabla sería enorme e inútil; el parcial se mantiene diminuto y siempre en memoria.

> [!danger] Cada índice ralentiza las escrituras
> Un `INSERT` actualiza la tabla **y todos sus índices**. Una tabla con 12 índices tiene escrituras varias veces más lentas.
> ```sql
> -- Índices que nadie usa: candidatos a borrar
> SELECT schemaname, relname, indexrelname, idx_scan
> FROM pg_stat_user_indexes WHERE idx_scan = 0 ORDER BY relname;
> ```

## 🐛 El N+1

```java
// ❌ 1 consulta + N consultas
List<Pedido> pedidos = repo.findAll();
pedidos.forEach(p -> p.getCliente().getNombre());     // lazy: una consulta por pedido

// ✅ Una sola
@Query("SELECT p FROM Pedido p JOIN FETCH p.cliente")
List<Pedido> findAllConCliente();

// ✅ O con grafo de entidades
@EntityGraph(attributePaths = {"cliente", "lineas"})
List<Pedido> findAll();
```

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 50     # agrupa las cargas lazy en lotes
    open-in-view: false                  # ← evita el N+1 durante la serialización
```

> [!danger] `open-in-view: true` es el default de Spring Boot y produce N+1 invisibles
> Mantiene la sesión de Hibernate abierta durante la serialización de la respuesta: cada relación lazy que Jackson toca dispara una consulta **fuera de la transacción**, en pleno render. Es el N+1 más difícil de detectar porque no aparece en el código del servicio.

```java
@Test
void el_endpoint_no_genera_N_mas_1() {
    var contador = new ContadorDeConsultas();
    api.get("/api/v1/pedidos");
    assertTrue(contador.total() < 5);      // no proporcional a la cantidad de filas
}
```

## 📖 Réplicas de lectura

```java
@Transactional(readOnly = true)     // → réplica
public List<MovimientoDto> extracto(CuentaId cuenta) { ... }

@Transactional                       // → primaria
public void transferir(ComandoTransferir cmd) { ... }
```

| Escala | No escala |
|---|---|
| lecturas (linealmente) | escrituras |
| reportes y dashboards | operaciones críticas |
| consultas analíticas | validaciones de negocio |

> [!danger] La réplica está atrasada
> Milisegundos en condiciones normales, segundos bajo carga. Nunca valides una operación crítica contra una réplica: el saldo que leés puede ser anterior a un retiro ya confirmado — ver [[02 - CAP PACELC y Consistencia]].

## 🧊 Desnormalización y vistas materializadas

```sql
-- Una consulta cara que se repite mucho
CREATE MATERIALIZED VIEW resumen_mensual AS
SELECT cuenta_id,
       date_trunc('month', fecha) AS mes,
       SUM(monto) FILTER (WHERE monto > 0) AS ingresos,
       SUM(monto) FILTER (WHERE monto < 0) AS egresos,
       COUNT(*) AS movimientos
FROM apunte a JOIN asiento s ON s.id = a.asiento_id
GROUP BY cuenta_id, date_trunc('month', fecha);

CREATE UNIQUE INDEX ON resumen_mensual(cuenta_id, mes);

-- Refrescar sin bloquear lecturas
REFRESH MATERIALIZED VIEW CONCURRENTLY resumen_mensual;
```

| Ventaja | Costo |
|---|---|
| La consulta pasa de segundos a milisegundos | los datos tienen la antigüedad del último refresco |
| Reduce carga de CPU en la base | ocupa espacio |
| Simple de agregar | hay que programar el refresco |

> [!tip] Una vista materializada es un caché que vive en la base
> Aplican las mismas preguntas de [[04 - Estrategias de Cache]]: ¿cuánta desactualización se tolera? ¿cuándo se refresca? ¿qué pasa si el refresco falla?
> Para reportes y dashboards suele ser la solución más simple y efectiva.

### Desnormalizar con criterio

```sql
-- El saldo se deriva de los apuntes, pero calcularlo cuesta con millones de filas
CREATE TABLE saldo_snapshot (
    cuenta_id     UUID PRIMARY KEY,
    saldo         NUMERIC(19,4) NOT NULL,
    ultimo_apunte BIGINT NOT NULL       -- desde dónde seguir sumando
);
```

> [!warning] La desnormalización es correcta solo si es reconstruible
> El snapshot tiene que poder regenerarse desde los datos originales y dar exactamente lo mismo. Si no, dejó de ser un caché y se convirtió en **una segunda fuente de verdad** que va a divergir — ver [[Fase 8 - Batch y Cierre de Dia]].

## 🗂️ Particionado local

```sql
CREATE TABLE apunte (
    id BIGSERIAL, cuenta_id UUID, monto NUMERIC(19,4), fecha TIMESTAMPTZ
) PARTITION BY RANGE (fecha);

CREATE TABLE apunte_2026_09 PARTITION OF apunte
    FOR VALUES FROM ('2026-09-01') TO ('2026-10-01');
```

| Beneficio | Detalle |
|---|---|
| Poda de particiones | una consulta por fecha solo lee la partición relevante |
| Mantenimiento | `VACUUM` y reindexado por partición |
| Archivado | `DETACH PARTITION` es instantáneo |
| Borrado masivo | `DROP` de partición en vez de `DELETE` de millones de filas |

> [!tip] Borrar una partición es instantáneo; borrar un millón de filas no
> `DELETE FROM apunte WHERE fecha < '2020-01-01'` genera un millón de tuplas muertas, dispara `VACUUM` y puede tardar horas. `DROP TABLE apunte_2019_12` tarda milisegundos.
> Si tenés retención por tiempo, particionar por fecha se paga solo.

## 🔌 Conexiones

```
conexiones_totales = instancias × maximum-pool-size ≤ límite de la base
```

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10
      connection-timeout: 3000
      max-lifetime: 1800000
      leak-detection-threshold: 60000
```

> [!warning] Un pool grande no acelera nada
> Una base con 8 núcleos no procesa 200 consultas en paralelo: las encola. Un pool grande solo mueve la cola del cliente al servidor, donde cada conexión además consume memoria.
> La fórmula de referencia de HikariCP es `núcleos × 2 + husos de disco` — bastante más chica de lo que la intuición sugiere.

Detalle en [[06 - Bases de Datos Gestionadas]].

## 🧰 Persistencia políglota

| Necesito | Motor |
|---|---|
| Transacciones e integridad | PostgreSQL |
| Búsqueda de texto completo | OpenSearch / AI Search |
| Caché y estructuras en memoria | Redis |
| Series temporales y métricas | Timescale / Prometheus |
| Analítico sobre grandes volúmenes | Redshift / Synapse |
| Clave-valor a escala enorme | DynamoDB / Cosmos DB |

> [!danger] Cada motor adicional es un sistema más que operar
> Respaldos, monitoreo, actualizaciones, experiencia en el equipo, y un modo de fallo nuevo. La regla es agregar un motor solo cuando el problema **no** se resuelve razonablemente con el que ya tenés.
> PostgreSQL cubre búsqueda de texto, JSON, geoespacial y series temporales razonablemente bien. Muchas veces "necesitamos Elasticsearch" en realidad era "necesitamos un índice GIN".

## 📊 Cuándo escalar qué

| Síntoma | Causa probable | Acción |
|---|---|---|
| Una consulta lenta | falta índice | índice |
| CPU alta con muchas consultas | consultas ineficientes | optimizar antes de agrandar |
| Muchas lecturas repetidas | falta caché | caché |
| Lecturas saturan la primaria | falta distribución | réplicas |
| Escrituras saturan | límite real de una instancia | particionar / shardear |
| Tabla enorme, consultas por fecha | falta poda | particionado local |
| Conexiones al tope | pool mal dimensionado | ajustar pool o agrupador |

## 🧪 Tests

```java
@Test
void la_consulta_del_extracto_usa_indice() {
    String plan = jdbc.queryForObject(
        "EXPLAIN SELECT * FROM apunte WHERE cuenta_id = ? ORDER BY id DESC LIMIT 50",
        String.class, cuentaId);
    assertFalse(plan.contains("Seq Scan"));
}

@Test
void el_endpoint_no_genera_N_mas_1() { ... }

@Test
void la_vista_materializada_coincide_con_el_calculo_directo() { ... }

@Test
void el_snapshot_reconstruido_da_el_mismo_resultado() { ... }

@Test
void el_extracto_de_una_cuenta_con_un_millon_de_movimientos_responde_bajo_200ms() { ... }
```

> [!tip] Probá con volumen realista
> Todo anda con 100 filas. Los problemas de índices, N+1 y paginación aparecen con millones. Un conjunto de datos de prueba con volumen real es lo que hace útiles a estos tests.

## ⚠️ Errores Comunes

> [!warning] Los que se pagan al crecer
> - **Shardear sin agotar los pasos anteriores.**
> - **Optimizar por tiempo medio** en vez de tiempo total.
> - **`open-in-view: true`**: N+1 invisibles en la serialización.
> - **Índices de más**: cada uno ralentiza las escrituras.
> - **Orden incorrecto en índices compuestos**: no se usan.
> - **Validar operaciones críticas contra una réplica.**
> - **Pool grande** creyendo que acelera.
> - **Desnormalizar sin poder reconstruir.**
> - **`DELETE` masivo** en vez de particionar y soltar particiones.
> - **Agregar un motor nuevo** por un problema que resolvía un índice.

## 📌 Notas Rápidas
- El orden de intervención (índices → caché → réplicas → instancia → desnormalizar → particionar → shardear) es lo más útil de esta nota. En una entrevista, recitarlo demuestra criterio: la respuesta a "¿cómo escalás la base?" no es "shardeo".
- `pg_stat_statements` ordenado por tiempo total es la primera consulta que correría en cualquier sistema con problemas de base. Casi siempre las tres primeras filas explican el 80% de la carga.
- El N+1 por `open-in-view` es de los bugs de rendimiento más comunes en Spring Boot y de los más fáciles de arreglar: una línea de configuración.

## 🔗 Relaciones
- **Módulo**: [[System Design]]
- **Anterior**: [[04 - Estrategias de Cache]]
- **Siguiente**: [[06 - Colas y Backpressure]]
- **Se conecta con**: [[03 - Indices y Queries Lentas]], [[12 - Particionado y Sharding]], [[01 - JPA e Hibernate]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

