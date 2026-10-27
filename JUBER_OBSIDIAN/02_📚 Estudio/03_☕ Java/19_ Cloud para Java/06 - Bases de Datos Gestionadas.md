---
tags:
  - Java
  - Cloud
  - Datos
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - Bases de Datos Java
Palabras clave:
  - RDS, Aurora, Azure SQL, Flexible Server, DynamoDB, Cosmos DB, pool, replicas, failover
---

## 🧠 Concepto

Una base gestionada te quita los parches, los respaldos, la replicación y el failover. **No te quita** diseñar el esquema, escribir consultas eficientes ni dimensionar las conexiones — y ahí es donde se concentran los problemas.

## 🗄️ El catálogo

| Necesito | Azure | AWS |
|---|---|---|
| PostgreSQL | Azure Database for PostgreSQL Flexible Server | RDS PostgreSQL · **Aurora PostgreSQL** |
| MySQL | Azure Database for MySQL Flexible Server | RDS MySQL · Aurora MySQL |
| SQL Server | **Azure SQL Database** | RDS SQL Server |
| NoSQL clave-valor | Cosmos DB | **DynamoDB** |
| Documental | Cosmos DB (API Mongo) | DocumentDB |
| Caché en memoria | Azure Cache for Redis | ElastiCache · MemoryDB |
| Analítico | Synapse | Redshift |

> [!tip] Aurora no es RDS
> RDS ejecuta el motor estándar sobre almacenamiento gestionado. **Aurora** reimplementa la capa de almacenamiento: replica en 6 copias entre 3 zonas, hace failover en segundos y escala lecturas con hasta 15 réplicas. Es compatible con Postgres y MySQL a nivel de protocolo.
> Cuesta más y, para cargas serias, suele valerlo.

### Elegir el motor

```
¿Datos relacionales con transacciones e integridad referencial?
├── SÍ → PostgreSQL gestionado (la opción por defecto razonable)
└── NO → ¿acceso por clave, latencia de milisegundo y escala enorme?
         ├── SÍ → DynamoDB / Cosmos DB
         └── NO → probablemente igual querías PostgreSQL
```

> [!warning] NoSQL no es "la base moderna"
> Es una base con un modelo de acceso distinto: rapidísima por clave, incómoda para todo lo demás. Si tu dominio tiene relaciones, transacciones y consultas variadas —como cualquier sistema financiero— **PostgreSQL es la respuesta correcta** y va a seguir siéndolo con mucho volumen.

## 🔌 El pool de conexiones

El problema número uno de Java contra bases gestionadas.

```
conexiones_totales = instancias × maximum-pool-size
```

```
20 instancias × pool de 10 = 200 conexiones solicitadas
Base gestionada pequeña      = 100 conexiones permitidas
                             → la mitad de las instancias no conecta 💥
```

> [!danger] El límite de conexiones depende del tamaño de la instancia
> En las bases gestionadas de Postgres el máximo escala con la memoria de la instancia: una instancia chica puede permitir ~50-100 conexiones y una grande varios miles. **Consultá el límite de tu tamaño concreto antes de escalar horizontalmente.**

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10          # × instancias ≤ límite de la base
      minimum-idle: 2
      connection-timeout: 3000       # falla rápido si no hay conexión
      idle-timeout: 600000
      max-lifetime: 1800000          # menor que cualquier timeout del servidor
      leak-detection-threshold: 60000
      validation-timeout: 3000
```

> [!tip] Un pool más grande no es más rápido
> Una base con 8 núcleos no procesa 200 consultas en paralelo: las encola. Un pool grande solo mueve la cola del cliente al servidor, donde cada conexión además consume memoria.
> La regla práctica de HikariCP es sorprendentemente chica: **`núcleos × 2 + husos de disco`**. Para la mayoría de los servicios, entre 5 y 15 por instancia.

### El agrupador externo

Cuando hay muchas instancias, la solución es un intermediario:

| | Azure | AWS |
|---|---|---|
| Agrupador gestionado | PgBouncer integrado en Flexible Server | **RDS Proxy** |
| Qué resuelve | miles de clientes → decenas de conexiones reales | ídem, y acelera el failover |

> [!warning] El modo de agrupación importa
> En modo *transaction*, una conexión se comparte entre clientes al terminar cada transacción. Eso rompe las sentencias preparadas del lado del servidor y las variables de sesión.
> Con PgBouncer en ese modo hay que desactivar la caché de sentencias preparadas del driver:
> ```
> jdbc:postgresql://host:5432/db?prepareThreshold=0
> ```

## 📖 Réplicas de lectura

```yaml
spring:
  datasource:
    primaria:
      url: jdbc:postgresql://primaria:5432/corebank
    replica:
      url: jdbc:postgresql://replica:5432/corebank
```

```java
@Transactional(readOnly = true)     // Spring rutea a la réplica si hay AbstractRoutingDataSource
public List<MovimientoDto> extracto(CuentaId cuenta) { ... }

@Transactional                       // escritura: siempre la primaria
public void transferir(ComandoTransferir cmd) { ... }
```

> [!danger] La réplica está atrasada — y eso cambia el resultado
> El lag es de milisegundos en condiciones normales y de **segundos o más** bajo carga. Nunca valides una operación crítica contra una réplica: el saldo que leés puede ser anterior a un retiro ya confirmado, y habilitás el doble gasto.
>
> | Consulta | De dónde |
> |---|---|
> | Validar saldo antes de transferir | **primaria** |
> | Extracto histórico | réplica ✅ |
> | Dashboard y reportes | réplica ✅ |
> | Saldo justo después de operar | primaria (read-your-writes) |

Ver [[02 - CAP PACELC y Consistencia]] para el detalle del compromiso.

## 🔄 Failover

```
Primaria cae → la nube promueve la réplica → el DNS apunta al nuevo primario
```

| | Tiempo típico |
|---|---|
| Azure SQL / Flexible Server con alta disponibilidad de zona | ~60 s |
| RDS Multi-AZ | 60-120 s |
| **Aurora** | ~30 s (a veces menos) |
| RDS Proxy | acelera notablemente la reconexión |

> [!warning] Tu aplicación tiene que sobrevivir al failover
> Durante la promoción, las conexiones existentes se cortan. Sin reintentos, **todas** las peticiones en vuelo fallan y el pool queda lleno de conexiones muertas.

```java
@Retryable(
    retryFor = { CannotGetJdbcConnectionException.class, TransientDataAccessException.class },
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000, multiplier = 2, random = true))
public void operacion() { ... }
```

```yaml
spring:
  datasource:
    hikari:
      # Detecta conexiones muertas antes de entregarlas
      connection-test-query: SELECT 1
      validation-timeout: 3000
      keepalive-time: 120000
```

> [!tip] Cuidado con la caché de DNS de la JVM
> El failover cambia a qué IP resuelve el nombre del servidor. Si la JVM cacheó la resolución, sigue intentando conectarse a la instancia caída.
> ```
> -Dnetworkaddress.cache.ttl=30
> -Dnetworkaddress.cache.negative.ttl=5
> ```

## 💾 Respaldos

| | Azure | AWS |
|---|---|---|
| Automáticos | ✅ retención configurable | ✅ retención configurable |
| Restauración a un punto en el tiempo | ✅ | ✅ |
| Copia entre regiones | ✅ | ✅ |
| Protección contra borrado | bloqueo de recurso | protección de eliminación |

> [!danger] Un respaldo que nunca se restauró no es un respaldo
> Es un archivo. La única forma de saber que sirve es **restaurarlo periódicamente** en un entorno aparte y verificar que los datos están completos. Es el ejercicio de recuperación más barato y el que más seguido falta — ver [[14 - Multiregion y DR]].

> [!warning] Los respaldos automáticos suelen borrarse con la instancia
> En varias configuraciones, eliminar la base elimina también sus respaldos automáticos. Para retención larga o cumplimiento, hacen falta copias explícitas con su propio ciclo de vida y protección contra borrado.

## 📊 Dimensionar

| Señal | Qué indica |
|---|---|
| CPU sostenida > 80% | falta capacidad o hay consultas ineficientes |
| Conexiones cerca del límite | pool mal dimensionado |
| Memoria disponible baja | poca caché: más lecturas a disco |
| IOPS al tope | almacenamiento subdimensionado |
| Latencia de consultas creciente | falta un índice, o crecieron los datos |
| Lag de réplica creciente | la réplica no da abasto |

> [!tip] Antes de agrandar la instancia, revisá las consultas
> Un índice faltante puede costar 100× más CPU que la consulta correcta. Agrandar la instancia para compensar una consulta mala es la forma más cara de resolver el problema, y hay que repetirla cada seis meses.
> Empezá por `pg_stat_statements` y `EXPLAIN ANALYZE` — ver [[03 - Indices y Queries Lentas]].

```sql
-- Las 10 consultas que más tiempo total consumen
SELECT calls, mean_exec_time, total_exec_time, query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

### Escalado

| Tipo | Cómo | Corte de servicio |
|---|---|---|
| Vertical | instancia más grande | breve (reinicio) |
| Réplicas de lectura | agregar réplicas | ninguno |
| Serverless | escala sola (Aurora Serverless, Azure SQL serverless) | ninguno |
| Sharding | [[12 - Particionado y Sharding]] | proyecto entero |

## 🌐 Red

```
❌ Base con endpoint público y contraseña
✅ Base sin acceso público, en la red privada
✅ Autenticación con identidad gestionada, sin contraseña
```

```java
// Azure: token de Entra ID en vez de contraseña
var token = new DefaultAzureCredentialBuilder().build()
    .getToken(new TokenRequestContext().addScopes("https://ossrdbms-aad.database.windows.net/.default"))
    .block();

// AWS: token IAM para RDS, válido 15 minutos
String token = RdsUtilities.builder().region(US_EAST_1).build()
    .generateAuthenticationToken(b -> b.hostname(host).port(5432).username(usuario));
```

> [!tip] Autenticación sin contraseña también para la base
> Es la extensión natural de [[05 - Identidad y Acceso]]: no hay contraseña que rotar ni que filtrar, y el acceso queda auditado en el plano de identidad.
> El detalle práctico: el token expira (15 minutos en RDS), así que el pool debe renovarlo — se resuelve con un `max-lifetime` menor a la expiración.

## 🧪 Tests

```java
@Test
void el_pool_no_excede_el_limite_de_la_base() {
    int total = instanciasEsperadas * config.getMaximumPoolSize();
    assertTrue(total <= limiteDeConexionesDeLaBase);
}

@Test
void tras_un_failover_la_app_se_recupera_sola() {
    toxiproxy.cortar("db");
    assertThrows(Exception.class, () -> repo.count());
    toxiproxy.restaurar("db");
    esperarQue(() -> repo.count() >= 0, Duration.ofSeconds(30));
}

@Test
void las_lecturas_de_solo_lectura_van_a_la_replica() { ... }

@Test
void la_validacion_de_saldo_lee_de_la_primaria() {
    simularLagDeReplica(Duration.ofSeconds(10));
    assertEquals(saldoReal, servicio.saldoParaValidacion(cuenta));
}

@Test
void ninguna_consulta_del_extracto_hace_seq_scan() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que tumban el servicio
> - **No calcular `instancias × pool`** contra el límite: la mitad no conecta.
> - **Pool enorme** creyendo que acelera: solo mueve la cola.
> - **`max-lifetime` mayor** al timeout del servidor o a la expiración del token.
> - **Validar operaciones críticas contra una réplica.**
> - **Sin reintentos**: cada failover es una caída completa.
> - **Caché de DNS de la JVM**: sigue apuntando a la instancia caída.
> - **Respaldos nunca restaurados.**
> - **Base con endpoint público.**
> - **Agrandar la instancia** en vez de arreglar la consulta.
> - **PgBouncer en modo transaction** sin desactivar sentencias preparadas.

## 📌 Notas Rápidas
- El cálculo `instancias × pool ≤ límite` es de las cosas más simples de este módulo y de las que más incidentes causa al escalar. Ponelo en un test.
- La base gestionada resuelve la operación, **no el diseño**. Un esquema mal normalizado y consultas sin índice cuestan igual —o más— en la nube, porque ahí el costo es explícito.
- Aurora vs RDS es una pregunta habitual de entrevista: la diferencia real está en la capa de almacenamiento distribuida, el failover más rápido y hasta 15 réplicas de lectura.
- Se aplica en [[Fase 1 - API de Cuentas]] y [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[05 - Identidad y Acceso]]
- **Siguiente**: [[07 - Mensajeria Gestionada]]
- **Se conecta con**: [[02 - JDBC y Connection Pools]], [[03 - Indices y Queries Lentas]], [[05 - Azure SQL y Blob Storage]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

