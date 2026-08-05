---
tags:
  - Java
  - NoSQL
Creado: 2026-05-21
Relacionado:
  - Bases de Datos Java
---

# NoSQL desde Java

NoSQL no significa "mejor que SQL". Significa otro modelo de datos para problemas distintos.

## Opciones

| Tipo | Ejemplo | Uso |
|---|---|---|
| Documentos | MongoDB | documentos flexibles |
| Key-value | Redis | cache, sesiones, locks |
| Wide-column | Cassandra | escritura masiva distribuida |
| Search | Elasticsearch | busqueda de texto |

## Preguntas antes de elegir

- Necesito joins?
- Necesito transacciones fuertes?
- El esquema cambia mucho?
- El patron principal es lectura por clave?
- Necesito busqueda textual?

## Spring Data Mongo ejemplo

```java
@Document("products")
public class ProductDocument {
    @Id
    private String id;
    private String name;
}
```

## Checklist

- [ ] Elijo por patron de acceso, no por moda.
- [ ] Entiendo consistencia y transacciones.
- [ ] Modelo documentos segun consultas reales.
- [ ] No duplico datos sin estrategia de sincronizacion.
