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

## 🔹 Arquitectura en Capas (Layered Architecture)

**Qué es:** Organiza el código en **capas horizontales**, cada una con una responsabilidad clara, donde cada capa solo puede depender de la capa inmediatamente inferior (o de abstracciones de ella). Es el patrón más común y el que casi todo backend usa aunque no lo llame explícitamente así.

```
┌─────────────────────────────────┐
│  Presentación (Controllers/API)  │  recibe HTTP, valida input, serializa output
├─────────────────────────────────┤
│  Negocio (Services)              │  lógica de negocio, orquesta casos de uso
├─────────────────────────────────┤
│  Acceso a Datos (Repositories)   │  persistencia, queries
├─────────────────────────────────┤
│  Base de Datos                   │
└─────────────────────────────────┘
```

**Regla de dependencia:** cada capa solo conoce a la de abajo, nunca a la de arriba. Un Repository no debe saber nada de HTTP; un Controller no debe tener SQL embebido.

```java
@RestController                              // Presentación
public class ProductoController {
    private final ProductoService service;    // depende de Negocio

    @GetMapping("/productos/{id}")
    public ProductoDTO obtener(@PathParam Long id) {
        return service.buscarPorId(id);
    }
}

@Service                                      // Negocio
public class ProductoService {
    private final ProductoRepository repository; // depende de Acceso a Datos

    public ProductoDTO buscarPorId(Long id) {
        Producto producto = repository.findById(id).orElseThrow();
        return ProductoDTO.from(producto);   // regla de negocio: mapear a DTO
    }
}

@Repository                                   // Acceso a Datos
public interface ProductoRepository extends JpaRepository<Producto, Long> {}
```

**Ventajas:**
- Simple de entender y de ubicar dónde va cada cosa — curva de aprendizaje baja para nuevos miembros del equipo.
- Separación clara de responsabilidades por capa.
- Es el punto de partida natural antes de evolucionar a Hexagonal/Clean Architecture si el dominio se vuelve complejo.

**Desventajas:**
- Con el tiempo, tiende al "anemic domain model": toda la lógica termina en Services, y las entidades son solo contenedores de datos sin comportamiento.
- Acoplamiento a frameworks/infraestructura si no se cuida (ej. la capa de negocio importando anotaciones de JPA directamente).
- No fuerza que el dominio sea independiente de la infraestructura — para eso existe Hexagonal (ver abajo).

**Casos de uso:** la gran mayoría de APIs CRUD, aplicaciones empresariales de complejidad media, el punto de partida por defecto salvo que haya una razón concreta para algo más elaborado.

## 🔹 Arquitectura Hexagonal (Ports and Adapters)

**Qué es:** Evolución de la arquitectura en capas que invierte la dependencia: el **dominio** (lógica de negocio) queda en el centro, sin conocer nada de infraestructura (frameworks, HTTP, base de datos). La comunicación con el exterior pasa por **puertos** (interfaces que el dominio define) implementados por **adaptadores** (la infraestructura real).

```
        Adaptador de entrada (REST Controller)
                    │
                    ▼
        Puerto de entrada (Use Case interface)
                    │
        ┌───────────▼───────────┐
        │   DOMINIO (núcleo)     │  ← no conoce Spring, JPA, HTTP
        │   entidades + reglas   │
        └───────────┬───────────┘
                    │
        Puerto de salida (Repository interface)
                    │
                    ▼
        Adaptador de salida (JPA Repository real)
```

Ver desarrollo completo con código en [[03 - Arquitectura Hexagonal]].

**Diferencia clave con Layered clásica:** en Layered, el Service (negocio) depende directamente de un Repository concreto (acoplado a JPA). En Hexagonal, el dominio define una **interfaz** (puerto de salida) y no sabe si detrás hay JPA, MongoDB o un mock — la dependencia se invierte (Dependency Inversion Principle aplicado a nivel arquitectónico).

**Ventajas:**
- El dominio se puede testear sin levantar Spring, JPA ni una base de datos real.
- Cambiar de proveedor (JPA → MongoDB, REST → gRPC) no toca el núcleo de negocio.
- Fuerza a pensar el dominio primero, la infraestructura después.

**Desventajas:**
- Más código (interfaces + implementaciones) para casos simples — sobre-ingeniería si el dominio es un CRUD trivial.
- Curva de aprendizaje mayor para el equipo.

**Casos de uso:** dominios de negocio complejos y de larga vida, donde el costo de la abstracción se paga con creces en mantenibilidad; no recomendado para un CRUD simple sin lógica de negocio real.

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
| En Capas (Layered) | N/A | Baja | Simple | Medio-Alto (entre capas adyacentes) |
| Hexagonal | N/A | Media-Alta | Simple | Bajo (dominio aislado de infraestructura) |

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

## Checklist Capas y Hexagonal
- [ ] Puedo dibujar y explicar las capas típicas (Presentación/Negocio/Datos) y la regla de dependencia entre ellas.
- [ ] Sé explicar qué es el "anemic domain model" y por qué la arquitectura en capas tiende a caer en eso.
- [ ] Puedo explicar puertos y adaptadores en Hexagonal, con un ejemplo propio (no solo el genérico).
- [ ] Sé argumentar cuándo Hexagonal vale la pena y cuándo es sobre-ingeniería para el caso.
- [ ] Entiendo cómo el Dependency Inversion Principle es lo que hace posible que el dominio no dependa de la infraestructura en Hexagonal.

## Relacionado
[[Postulacion|Senior Java Developer - Coforge/BCP]] — pide explícitamente estos dos patrones como requisito indispensable.