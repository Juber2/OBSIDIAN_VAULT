---
tags:
  - Java
  - NoSQL
Creado: 2026-05-21
Actualizado: 2026-08-12
Relacionado:
  - Bases de Datos Java
---

# NoSQL desde Java

NoSQL no significa "mejor que SQL". Significa otro modelo de datos para problemas distintos, con otros trade-offs de consistencia.

## Opciones

| Tipo | Ejemplo | Uso |
|---|---|---|
| Documentos | MongoDB | documentos flexibles, esquema variable |
| Key-value | Redis | cache, sesiones, locks |
| Wide-column | Cassandra | escritura masiva distribuida |
| Search | Elasticsearch | búsqueda de texto |

## ACID vs BASE (concepto central, muy preguntado en entrevista)

| | ACID (SQL tradicional) | BASE (la mayoría de NoSQL) |
|---|---|---|
| **A**tomicity / **B**asically **A**vailable | transacción todo-o-nada | el sistema prioriza estar disponible, aunque sea con datos no 100% actualizados |
| **C**onsistency / **S**oft state | todos ven el mismo dato al instante | el estado puede cambiar con el tiempo sin nueva escritura (replicación en curso) |
| **I**solation / **E**ventual consistency | transacciones concurrentes no se pisan | tarde o temprano, todas las réplicas convergen al mismo valor |
| **D**urability | una vez confirmado, el dato persiste | igual aplica, pero el foco no está ahí |

**La idea de fondo:** ACID prioriza **consistencia** sobre disponibilidad; BASE prioriza **disponibilidad y escalabilidad horizontal** sobre consistencia inmediata. Esto conecta directo con el **teorema CAP**: en presencia de una partición de red (P), un sistema distribuido elige entre Consistencia (C) o Disponibilidad (A), no ambas a la vez.

```
CAP: Consistency, Availability, Partition tolerance — elegís 2 de 3
     (en la práctica, P es obligatorio en sistemas distribuidos, así que la elección real es C vs A)

MongoDB / Cassandra → tienden a AP (disponibilidad, consistencia eventual)
Bases SQL tradicionales (single-node) → CA (consistencia fuerte, sin tolerancia real a partición)
```

**Ejemplo concreto:** si escribís en MongoDB con replicación, y consultás inmediatamente una réplica secundaria, podés leer el dato **anterior** durante una ventana breve (consistencia eventual) — no es un bug, es el trade-off que elegiste al usar ese modelo para ganar disponibilidad/escala.

## Modelado: "tablas" en NoSQL (colecciones/documentos)

En MongoDB no hay tablas con schema fijo — hay **colecciones** de documentos. La regla de modelado cambia respecto a SQL: en vez de normalizar (evitar duplicación), a menudo conviene **desnormalizar** pensando en el patrón de consulta.

```java
// SQL: pedido y items en tablas separadas con JOIN
// NoSQL: documento embebido, pensado para leerse de una sola consulta
@Document("pedidos")
public class Pedido {
    @Id
    private String id;
    private String clienteId;
    private List<ItemPedido> items;   // embebido, no en colección aparte
    private BigDecimal total;
}
```

Regla práctica: **modelá según cómo vas a consultar, no según cómo normalizarías en SQL.** Si siempre leés el pedido completo con sus items juntos, embeber es más rápido que hacer un JOIN (que NoSQL no soporta de forma nativa y eficiente).

## Índices en NoSQL

Igual de importantes que en SQL — sin índice, una consulta por un campo que no es `_id` hace un scan completo de la colección.

```java
@Document("products")
public class ProductDocument {
    @Id
    private String id;

    @Indexed
    private String category;   // consultas frecuentes por categoría → índice

    @Indexed(unique = true)
    private String sku;        // único, como una unique constraint en SQL
}
```

```javascript
// equivalente en shell de Mongo
db.products.createIndex({ category: 1 })
db.products.createIndex({ sku: 1 }, { unique: true })
```

Tipos de índices comunes en MongoDB: simples (un campo), compuestos (varios campos, importa el orden), y de texto (para búsqueda full-text, aunque para eso Elasticsearch suele ser mejor opción dedicada).

## Consultas desde Java con Spring Data MongoDB

```java
public interface ProductRepository extends MongoRepository<ProductDocument, String> {
    List<ProductDocument> findByCategory(String category);
    Optional<ProductDocument> findBySku(String sku);
}
```

Consultas más complejas con `Query`/`Criteria`:
```java
Query query = new Query();
query.addCriteria(Criteria.where("category").is("electronica")
        .and("precio").gte(new BigDecimal("50")));
List<ProductDocument> resultado = mongoTemplate.find(query, ProductDocument.class);
```

## Transacciones en NoSQL (sí existen, con matices)

MongoDB soporta transacciones ACID multi-documento desde la versión 4.0, pero tienen más costo de performance que en SQL — se usan cuando de verdad se necesita atomicidad entre varios documentos, no por defecto en cada operación.

```java
mongoTemplate.execute(session -> {
    session.startTransaction();
    try {
        // operaciones...
        session.commitTransaction();
    } catch (Exception e) {
        session.abortTransaction();
        throw e;
    }
    return null;
});
```

## Preguntas antes de elegir SQL vs NoSQL

- ¿Necesito joins complejos entre entidades? → SQL.
- ¿Necesito transacciones fuertes multi-entidad como regla, no excepción? → SQL.
- ¿El esquema cambia mucho y seguido? → NoSQL (documentos).
- ¿El patrón principal es lectura por clave, alto volumen? → Key-value (Redis) o Wide-column (Cassandra).
- ¿Necesito búsqueda textual avanzada? → Elasticsearch.
- ¿Necesito escalar escritura horizontalmente por encima de todo? → Wide-column o documentos con sharding.

## Errores comunes

- Elegir NoSQL "porque es más moderno" sin que el patrón de acceso lo justifique.
- Modelar en MongoDB igual que en SQL (todo normalizado en colecciones separadas), perdiendo la ventaja de embeber y terminando haciendo "joins" manuales en el código de la app.
- No crear índices en NoSQL asumiendo que "es rápido por diseño" — sin índice, un `find` por un campo no indexado escanea toda la colección igual que en SQL.
- Confundir consistencia eventual con "el sistema está roto" al ver datos desactualizados momentáneamente en una réplica.

## Checklist
- [ ] Puedo explicar ACID vs BASE con un ejemplo concreto, no solo la sigla.
- [ ] Entiendo el teorema CAP y dónde se ubican MongoDB/Cassandra vs SQL tradicional.
- [ ] Sé modelar pensando en el patrón de consulta (embeber vs referenciar) en vez de normalizar por costumbre.
- [ ] Sé crear índices simples y compuestos, y explicar por qué importan igual que en SQL.
- [ ] Sé cuándo NoSQL sí soporta transacciones y cuál es el trade-off de usarlas.
- [ ] Elijo SQL vs NoSQL por patrón de acceso, no por moda.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

