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