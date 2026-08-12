# Patrones Arquitectónicos de Software

![[1742435149294.gif|500]]

## 🔹 Arquitectura Monolítica

**Qué es:** Toda la aplicación es una sola unidad desplegable.

| Aspecto | Detalle |
|---------|---------|
| Estructura | Un solo código base, un solo despliegue |
| Base de datos | Compartida |
| Comunicación | Llamadas directas en memoria |

**Ventajas:**
- Desarrollo inicial rápido y simple
- Fácil de debuggear y testear
- Una sola transacción abarca toda la lógica
- Menor complejidad operacional

**Desventajas:**
- Difícil de escalar componentes individuales
- Despliegues arriesgados (todo o nada)
- Código acoplado tiende a degradarse
- Un fallo puede tumbar todo el sistema

**Casos de uso:** MVPs, startups, aplicaciones pequeñas/medianas, equipos pequeños

---

## 🔹 Arquitectura de Microservicios

**Qué es:** Aplicación dividida en servicios independientes, cada uno con responsabilidad específica.

| Aspecto | Detalle |
|---------|---------|
| Estructura | Múltiples servicios autónomos |
| Base de datos | Cada servicio tiene la suya (idealmente) |
| Comunicación | HTTP/REST, gRPC, mensajería asíncrona |

**Ventajas:**
- Escalado independiente por servicio
- Despliegues independientes
- Equipos autónomos por dominio
- Tecnología heterogénea (cada servicio elige su stack)
- Aislamiento de fallos

**Desventajas:**
- Complejidad operacional alta
- Transacciones distribuidas son difíciles
- Latencia de red entre servicios
- Requiere DevOps maduro

**Tecnologías comunes:** Docker, Kubernetes, API Gateway, Service Mesh, RabbitMQ/Kafka

**Casos de uso:** Aplicaciones grandes, equipos múltiples, alta escalabilidad requerida

---

## 🔹 Event-Driven Architecture (EDA)

**Qué es:** Componentes se comunican mediante eventos de forma asíncrona y desacoplada.

| Aspecto | Detalle |
|---------|---------|
| Comunicación | Publicar/Suscribir (Pub/Sub) |
| Acoplamiento | Bajo (productores no conocen consumidores) |
| Procesamiento | Asíncrono |

**Componentes clave:**
- **Event Producer** → genera eventos
- **Event Broker** → distribuye eventos (Kafka, RabbitMQ, Azure Service Bus)
- **Event Consumer** → reacciona a eventos

**Ventajas:**
- Alto desacoplamiento
- Escalabilidad horizontal
- Resiliencia (colas persisten mensajes)
- Ideal para procesos largos o asincrónicos

**Desventajas:**
- Debugging complejo (flujo no lineal)
- Consistencia eventual (no inmediata)
- Orden de eventos puede ser problema
- Duplicados requieren idempotencia

**Casos de uso:** IoT, sistemas reactivos, microservicios, procesamiento de streams, CQRS + Event Sourcing

---

## 🔹 MVC (Model-View-Controller)

**Qué es:** Patrón que separa la aplicación en tres responsabilidades claras.

| Componente | Responsabilidad |
|------------|-----------------|
| **Model** | Datos y lógica de negocio |
| **View** | Interfaz de usuario |
| **Controller** | Coordina entrada del usuario y actualiza Model/View |

**Flujo:**
```
Usuario → Controller → Model → View → Usuario
```

**Ventajas:**
- Separación clara de responsabilidades
- Facilita testing (lógica separada de UI)
- Múltiples vistas para el mismo modelo
- Estándar ampliamente conocido

**Desventajas:**
- Controllers pueden volverse "fat" (demasiada lógica)
- En frontend moderno, el patrón ha evolucionado (MVVM, Flux, etc.)

**Frameworks que lo usan:** ASP.NET MVC, Spring MVC, Laravel, Ruby on Rails, Angular (variante)

**Casos de uso:** Aplicaciones web tradicionales, APIs con estructura clara

---

## 🔹 Master-Slave (Primary-Replica)

**Qué es:** Un nodo maestro coordina y delega trabajo a nodos esclavos.

| Nodo | Rol |
|------|-----|
| **Master (Primary)** | Escribe, coordina, distribuye tareas |
| **Slave (Replica)** | Lee, procesa tareas delegadas |

**Ventajas:**
- Escala lecturas fácilmente (múltiples réplicas)
- Redundancia y alta disponibilidad
- Distribución de carga

**Desventajas:**
- Escrituras centralizadas (cuello de botella)
- Replicación puede tener lag
- Failover del master requiere estrategia
- Consistencia eventual entre nodos

**Casos de uso:** Bases de datos distribuidas (MySQL replication, PostgreSQL streaming), procesamiento paralelo, sistemas de caché

---

## 🔹 SAGA (Transacciones Distribuidas)

**Qué es:** Patrón para mantener consistencia de datos entre varios microservicios sin usar una transacción ACID distribuida (que en microservicios no existe de forma práctica — cada servicio tiene su propia base de datos). Una operación de negocio se parte en una secuencia de transacciones locales, cada una con su **acción de compensación** para deshacer el efecto si un paso posterior falla.

```
Pedido creado → Reservar stock → Cobrar pago → Confirmar envío
     ✅              ✅              ❌ falla
                                       ↓
                        Compensar: liberar stock reservado
```

### Dos formas de implementarlo

| Estilo | Cómo funciona | Trade-off |
|---|---|---|
| **Coreografía** | cada servicio escucha eventos y reacciona, sin coordinador central (Event-Driven puro) | bajo acoplamiento, pero difícil de rastrear el flujo completo al crecer |
| **Orquestación** | un orquestador central (ej. un Saga Orchestrator) le dice a cada servicio qué hacer y en qué orden | flujo explícito y fácil de rastrear, pero el orquestador es un punto más de complejidad/acoplamiento |

**Ejemplo con coreografía (eventos):**
```
PedidoService   → publica PedidoCreado
InventarioService → escucha PedidoCreado, reserva stock, publica StockReservado (o StockRechazado)
PagoService     → escucha StockReservado, cobra, publica PagoConfirmado (o PagoFallido)
InventarioService → si escucha PagoFallido, ejecuta compensación: libera el stock
```

**Ventajas:**
- Evita transacciones distribuidas de dos fases (2PC), que no escalan bien entre microservicios.
- Cada servicio mantiene su autonomía y su propia base de datos.

**Desventajas:**
- Consistencia **eventual**, no inmediata — hay una ventana donde el sistema está en estado intermedio.
- Cada paso necesita una compensación bien diseñada — no siempre es trivial (ej. "descontar dinero" no siempre se puede "deshacer" limpio, a veces requiere una operación compensatoria distinta, como un reembolso).
- Requiere idempotencia en cada handler (los eventos pueden reintentarse/duplicarse).

**Casos de uso:** checkout de e-commerce, reservas (vuelos + hotel + auto), cualquier flujo de negocio que cruza varios microservicios con su propia base de datos.

## 🔹 Cache Aside (Lazy Loading)

**Qué es:** Patrón de caching donde la **aplicación** (no la base de datos ni un proxy) es responsable de leer/escribir en el cache. Es el patrón de cache más común en la práctica.

```
Lectura:
  App → ¿está en cache? → Sí → devuelve desde cache
                        → No → lee de la BD → guarda en cache → devuelve

Escritura/actualización:
  App → escribe en la BD → invalida (borra) la entrada del cache
```

```java
public Producto obtenerProducto(Long id) {
    Producto cacheado = cache.get("producto:" + id);
    if (cacheado != null) {
        return cacheado;
    }
    Producto producto = productoRepository.findById(id);
    cache.put("producto:" + id, producto, Duration.ofMinutes(10));
    return producto;
}

public void actualizarProducto(Producto producto) {
    productoRepository.save(producto);
    cache.evict("producto:" + producto.getId());   // invalidar, no actualizar el cache directo
}
```

**Por qué invalidar y no actualizar el cache directo en el write:** actualizar el cache en cada escritura es más propenso a inconsistencias bajo concurrencia (dos escrituras simultáneas pueden dejar el cache con datos viejos). Invalidar es más simple y seguro: la próxima lectura repuebla el cache con el dato correcto.

**Ventajas:**
- Solo se cachea lo que realmente se lee (a diferencia de precargar todo).
- Resiliente a caídas del cache: si Redis está caído, la app sigue funcionando yendo directo a la BD (más lento, pero no se rompe).
- Simple de implementar sobre cualquier base de datos existente.

**Desventajas:**
- Primera lectura después de una invalidación es más lenta ("cache miss").
- Riesgo de datos "stale" (viejos) si la invalidación falla o hay una ventana de carrera entre escritura e invalidación.
- No sirve tal cual para datos que cambian constantemente (el hit ratio sería bajo).

**Comparación rápida con otras estrategias de cache:**

| Estrategia | Quién actualiza el cache | Cuándo conviene |
|---|---|---|
| Cache Aside | la app, bajo demanda (lazy) | caso general, la mayoría de los sistemas |
| Write-Through | se escribe al cache y a la BD en la misma operación | cuando la consistencia inmediata cache↔BD importa más que la latencia de escritura |
| Write-Behind | se escribe al cache, y a la BD de forma asíncrona/batch | alto volumen de escrituras, se tolera perder algo en un fallo |

**Casos de uso:** catálogos de productos, perfiles de usuario, cualquier dato de lectura frecuente y escritura poco frecuente — el par Redis + Cache Aside es prácticamente el default en arquitecturas de microservicios.

---

## 🔄 Comparativa Rápida

| Patrón | Escala | Complejidad | Despliegue | Acoplamiento |
|--------|--------|-------------|------------|--------------|
| Monolítico | Vertical | Baja | Simple | Alto |
| Microservicios | Horizontal | Alta | Independiente | Bajo |
| Event-Driven | Horizontal | Alta | Independiente | Muy bajo |
| MVC | N/A | Baja | Simple | Medio |
| Master-Slave | Lecturas | Media | Coordinado | Medio |

---

## 🎯 Cuándo usar cada uno

> [!tip] Regla general
> - **Empezando, equipo pequeño** → Monolítico
> - **Escala, equipos múltiples, dominios claros** → Microservicios
> - **Alta concurrencia, desacoplamiento, tiempo real** → Event-Driven
> - **Aplicación web con UI** → MVC (o variantes modernas)
> - **Base de datos con muchas lecturas** → Master-Slave

> [!note] Combinaciones comunes
> Los patrones no son excluyentes. Es común ver:
> - Microservicios + Event-Driven
> - MVC dentro de un Monolítico
> - Master-Slave en la capa de datos de cualquier arquitectura
> - Microservicios + SAGA (para transacciones cruzadas) + Cache Aside (para lecturas frecuentes)

## Checklist SAGA y Cache Aside
- [ ] Puedo explicar por qué no se usa una transacción ACID distribuida entre microservicios.
- [ ] Sé diferenciar SAGA por coreografía de SAGA por orquestación, con trade-offs de cada una.
- [ ] Entiendo qué es una acción de compensación y por qué debe ser idempotente.
- [ ] Sé implementar Cache Aside (lectura con fallback a BD, invalidación en escritura).
- [ ] Puedo justificar por qué invalidar el cache en vez de actualizarlo directo en cada escritura.

## Relacionado
[[Postulacion|Senior Java Developer - Coforge/BCP]] — pide explícitamente estos dos patrones como requisito indispensable.