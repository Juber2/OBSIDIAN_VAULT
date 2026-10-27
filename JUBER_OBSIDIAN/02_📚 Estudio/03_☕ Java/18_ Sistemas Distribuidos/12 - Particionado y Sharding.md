---
tags:
  - Java
  - Distribuido
  - Datos
Creado: 2026-09-08
Relacionado:
  - Sistemas Distribuidos
  - 05 - Base de Datos a Escala
Palabras clave:
  - sharding, particionado, hash consistente, hot partition, rebalanceo, clave de particion
---

## 🧠 Concepto

**Particionar** es dividir un conjunto de datos en subconjuntos que se guardan y procesan por separado.

| Término | Alcance |
|---|---|
| **Particionado** | dentro de una misma base (tablas particionadas de Postgres) |
| **Sharding** | entre bases o nodos distintos |

La diferencia práctica es enorme: el particionado local es transparente y barato; el sharding cambia el modelo de programación.

> [!danger] El sharding es el último recurso
> Antes de shardear, agotá: índices correctos, réplicas de lectura, caché, archivado de datos históricos, particionado local, y una instancia más grande. El sharding rompe las transacciones, los `JOIN` y la unicidad global. **Es la decisión más difícil de revertir de todo el módulo.**

## 🔑 La clave de partición

La decisión más importante. Determina dónde vive cada dato, y con eso qué consultas son rápidas y cuáles imposibles.

### Estrategias

| Estrategia | Cómo | Ventaja | Problema |
|---|---|---|---|
| **Por rango** | A-F → nodo 1, G-M → nodo 2 | consultas por rango eficientes | desbalance fácil |
| **Por hash** | `hash(clave) % N` | reparto uniforme | pierde el rango; rebalanceo caro |
| **Hash consistente** | anillo con nodos virtuales | rebalanceo mínimo | más complejo |
| **Por directorio** | tabla de ubicaciones | máxima flexibilidad | punto único de consulta |

```java
// ❌ Módulo simple: agregar un nodo remapea CASI TODO
int nodo = Math.abs(clave.hashCode()) % cantidadDeNodos;
// de 4 a 5 nodos → se mueve ~80% de los datos
```

### Hash consistente

```java
public class AnilloConsistente {

    private final SortedMap<Long, String> anillo = new TreeMap<>();
    private static final int REPLICAS_VIRTUALES = 150;

    public void agregarNodo(String nodo) {
        for (int i = 0; i < REPLICAS_VIRTUALES; i++) {
            anillo.put(hash(nodo + "#" + i), nodo);
        }
    }

    public String nodoPara(String clave) {
        if (anillo.isEmpty()) throw new IllegalStateException("anillo vacío");

        long h = hash(clave);
        SortedMap<Long, String> cola = anillo.tailMap(h);
        return cola.isEmpty() ? anillo.get(anillo.firstKey())    // vuelta al inicio
                              : cola.get(cola.firstKey());
    }
}
```

> [!tip] Los nodos virtuales resuelven dos problemas a la vez
> Sin ellos, cada nodo ocupa una sola posición del anillo y el reparto queda desbalanceado por azar. Con 150 réplicas virtuales por nodo, el reparto se empareja **y** al quitar un nodo su carga se distribuye entre todos los demás en vez de caer entera sobre el vecino.
> Al agregar o quitar un nodo se mueve aproximadamente `1/N` de los datos, no el 80%.

## 🔥 Las particiones calientes

El fallo más común: una partición recibe muchísimo más tráfico que las demás.

```
Clave = país
  PE: 60% del tráfico    ← esta partición se satura
  CL: 15%
  AR: 15%
  otros: 10%
```

| Causa | Ejemplo |
|---|---|
| Clave de baja cardinalidad | país, estado, categoría |
| Distribución sesgada | un cliente concentra el 40% de las operaciones |
| Clave temporal | todo lo de hoy en la misma partición |
| Clave secuencial | los identificadores nuevos van todos al mismo lado |

### Mitigaciones

```java
// 1 · Clave compuesta: más cardinalidad
String clave = clienteId + ":" + cuentaId;

// 2 · Salting para claves que se sabe calientes
String clave = esClienteGrande(id)
    ? id + ":" + ThreadLocalRandom.current().nextInt(10)   // se reparte en 10
    : id;
```

> [!warning] El salting rompe el orden dentro de la clave
> Si repartís los eventos de una cuenta en 10 sub-particiones, dejan de estar ordenados entre sí. Solo se aplica cuando el orden **no** importa para esa clave — típicamente en agregaciones o conteos, no en movimientos de dinero.

```sql
-- 3 · Particionado por tiempo Y por hash, para evitar la partición "de hoy"
CREATE TABLE apunte (...) PARTITION BY RANGE (fecha);

CREATE TABLE apunte_2026_09 PARTITION OF apunte
    FOR VALUES FROM ('2026-09-01') TO ('2026-10-01')
    PARTITION BY HASH (cuenta_id);        -- subparticionado

CREATE TABLE apunte_2026_09_0 PARTITION OF apunte_2026_09
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
```

## 💔 Lo que se pierde al shardear

### Las transacciones

```java
// Si origen y destino están en shards distintos, no hay transacción posible
@Transactional
public void transferir(CuentaId origen, CuentaId destino, Dinero monto) {
    cuentas.debitar(origen, monto);     // shard 1
    cuentas.acreditar(destino, monto);  // shard 3  💥 otra base
}
```

Las opciones son las de siempre: SAGA con compensación ([[06 - SAGA]]), o **elegir la clave de partición para que lo relacionado quede junto**.

> [!tip] La mejor clave de partición es la que evita las consultas entre shards
> Si particionás las cuentas por `cliente_id`, todas las cuentas de un cliente caen en el mismo shard y una transferencia entre cuentas propias sigue siendo una transacción local. Solo las transferencias a terceros cruzan.
> **Elegir bien la clave elimina la mayoría de los problemas antes de que existan.**

### Las consultas que cruzan shards

```java
// ❌ Consultar todos los shards y unir en memoria: lento y frágil
public List<Cuenta> buscarPorNombre(String nombre) {
    return shards.stream()
        .flatMap(s -> s.buscarPorNombre(nombre).stream())    // N consultas
        .toList();
}

// ✅ Un índice secundario global (motor de búsqueda o tabla de directorio)
public List<Cuenta> buscarPorNombre(String nombre) {
    List<UUID> ids = indiceGlobal.buscar(nombre);            // 1 consulta
    return cargarAgrupadoPorShard(ids);                      // solo los shards necesarios
}
```

Y la paginación con orden se vuelve genuinamente difícil: para devolver "los 20 más recientes" globalmente hay que traer 20 de cada shard y fusionar.

### La unicidad global

```sql
-- Esto solo garantiza unicidad DENTRO del shard
CREATE UNIQUE INDEX idx_documento ON cliente(documento);
```

Dos clientes con el mismo documento pueden existir en shards distintos. Las salidas: incluir la clave de partición en la restricción, o mantener un servicio de unicidad aparte.

## ⚖️ El rebalanceo

Al agregar capacidad hay que mover datos sin cortar el servicio.

```
1 · Doble escritura      escribir en el shard viejo Y en el nuevo
2 · Copia de fondo       migrar los datos históricos por lotes
3 · Verificación         comparar ambos lados
4 · Cambio de lectura    empezar a leer del nuevo
5 · Apagar el viejo      dejar de escribir y liberar
```

> [!warning] Nunca "pausar el servicio y mover los datos"
> Con volúmenes reales la migración tarda horas. La doble escritura permite migrar en caliente y **volver atrás en cualquier momento** si algo sale mal — que es la propiedad que realmente importa.

Una alternativa que simplifica mucho: **particiones fijas**. Se crean 1024 particiones lógicas desde el principio y se asignan a los nodos físicos; agregar un nodo es reasignar particiones, no recalcular hashes. Es lo que hacen Kafka y varias bases distribuidas.

## 🎯 Antes de shardear

| Alternativa | Ganancia típica |
|---|---|
| Índices correctos | 10-1000× en consultas |
| Réplicas de lectura | escala lecturas linealmente |
| Caché | quita el 80-95% de las lecturas |
| Archivar datos históricos | reduce el conjunto activo |
| Particionado local | mejora consultas por rango |
| Instancia más grande | 2-10× por bastante menos complejidad |

> [!danger] La pregunta antes de shardear
> *"¿Ya agoté todo lo anterior, con métricas que lo demuestren?"* Si la respuesta no es un sí con números, no shardees. La complejidad que agrega es permanente y afecta a todo el equipo, para siempre.

## 🧪 Tests

```java
@Test
void la_distribucion_es_uniforme() {
    var conteo = new HashMap<String, Integer>();
    IntStream.range(0, 100_000)
        .forEach(i -> conteo.merge(anillo.nodoPara("clave-" + i), 1, Integer::sum));

    int esperado = 100_000 / anillo.cantidadDeNodos();
    conteo.values().forEach(c ->
        assertTrue(Math.abs(c - esperado) < esperado * 0.15));   // ±15%
}

@Test
void agregar_un_nodo_mueve_aproximadamente_1_sobre_n() {
    var antes = mapear(claves);
    anillo.agregarNodo("nodo-5");
    var despues = mapear(claves);

    long movidas = claves.stream().filter(k -> !antes.get(k).equals(despues.get(k))).count();
    assertTrue(movidas < claves.size() * 0.30);      // no el 80% del módulo simple
}

@Test
void las_cuentas_de_un_cliente_caen_en_el_mismo_shard() {
    var shards = cuentasDe(clienteId).stream().map(router::shardPara).distinct().toList();
    assertEquals(1, shards.size());
}

@Test
void la_transferencia_entre_cuentas_propias_no_cruza_shards() { ... }

@Test
void ninguna_particion_recibe_mas_del_doble_de_la_media() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que son caros de revertir
> - **Shardear sin agotar las alternativas.**
> - **`hash % N`** en vez de hash consistente: agregar un nodo remapea casi todo.
> - **Clave de baja cardinalidad**: particiones calientes garantizadas.
> - **Clave temporal sin subparticionar**: todo el tráfico de hoy en una partición.
> - **No considerar las consultas al elegir la clave**: todo termina cruzando shards.
> - **Suponer unicidad global** con un índice único por shard.
> - **Rebalancear con corte de servicio.**
> - **Consultar todos los shards y unir en memoria** como estrategia habitual.
> - **Salting sobre claves donde el orden importa.**

## 📌 Notas Rápidas
- La clave de partición se elige **por las consultas**, no por los datos. La pregunta correcta es "¿qué consultas tienen que ser rápidas?" y después "¿qué clave las mantiene dentro de un shard?".
- Cambiar la clave de partición después es una migración de meses. Es la decisión que más conviene pensar despacio.
- Kafka usa el mismo concepto con otro vocabulario: la clave del mensaje determina la partición, y las particiones calientes existen igual. Lo que aprendas acá se transfiere directo — ver [[04 - Garantias de Entrega]].
- Vitess, Citus y las bases distribuidas nativas (CockroachDB, YugabyteDB) resuelven parte de esto por vos. Vale evaluarlas antes de shardear a mano.

## 🔗 Relaciones
- **Módulo**: [[Sistemas Distribuidos]]
- **Anterior**: [[11 - Relojes y Causalidad]]
- **Siguiente**: [[13 - Consenso y Raft]]
- **Se conecta con**: [[05 - Base de Datos a Escala]], [[03 - Indices y Queries Lentas]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

