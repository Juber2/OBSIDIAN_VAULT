# Tipos de Comunicación entre Servicios

![[1747541066021.gif|400]]
## 📦 SOAP (Simple Object Access Protocol)

**Protocolo:** XML sobre HTTP

| Aspecto | Detalle |
|---------|---------|
| Formato | XML puro |
| Contrato | WSDL (Web Services Description Language) |
| Estándares | WS-Security, WS-Transaction, WS-ReliableMessaging |

**Ventajas:**
- Seguridad robusta integrada
- Transacciones distribuidas
- Tipado fuerte con esquemas XSD

**Desventajas:**
- Verboso y pesado
- Rígido, difícil de evolucionar
- Parsing XML costoso

**Casos de uso:** Banca, sistemas legacy, integraciones enterprise B2B

---
## 🌐 REST (Representational State Transfer)

**Protocolo:** HTTP con recursos y verbos estándar

| Verbo | Operación |
|-------|-----------|
| GET | Leer |
| POST | Crear |
| PUT | Actualizar completo |
| PATCH | Actualizar parcial |
| DELETE | Eliminar |

**Ventajas:**
- Simple y ampliamente adoptado
- Cacheable por defecto
- Stateless, escala fácilmente
- Cualquier formato (JSON, XML, etc.)

**Desventajas:**
- Overfetching (traes más datos de los necesarios)
- Underfetching (necesitas múltiples requests)
- Sin contrato formal estándar (OpenAPI es opcional)

**Casos de uso:** APIs públicas, CRUD clásico, aplicaciones web

---
## 🧠 GraphQL

![[1741310665124.gif|400]]
**Protocolo:** HTTP POST a un único endpoint

**Ventajas:**
- Cliente define exactamente qué datos necesita
- Un solo request para datos relacionados
- Tipado fuerte con schema
- Introspection (autodocumentado)

**Desventajas:**
- Curva de aprendizaje
- Caché más complejo (no usa HTTP caching)
- N+1 queries si no se optimiza (DataLoader)
- Seguridad: queries maliciosas pueden ser costosas

**Casos de uso:** Apps móviles, dashboards complejos, BFF (Backend for Frontend)

---
## ⚡ gRPC

**Protocolo:** HTTP/2 con Protobuf (binario)

**Ventajas:**
- Ultra rápido y eficiente en red
- Streaming bidireccional nativo
- Contrato estricto con `.proto`
- Generación de código automática

**Desventajas:**
- No legible por humanos
- Soporte limitado en navegadores (necesita grpc-web)
- Debugging más complejo

**Casos de uso:** Comunicación interna entre microservicios, sistemas de alto rendimiento, IoT

---
## 📣 WebSocket

**Protocolo:** TCP con upgrade desde HTTP

**Ventajas:**
- Comunicación bidireccional persistente
- Baja latencia
- Eventos en tiempo real sin polling

**Desventajas:**
- Conexiones abiertas consumen recursos
- Manejo de reconexiones manual
- No cacheable
- Escalado horizontal más complejo (sticky sessions o pub/sub)

**Casos de uso:** Chat, juegos online, trading en vivo, notificaciones push

---
## 📬 Webhook

**Protocolo:** HTTP POST desde sistema externo hacia tu servidor

**Ventajas:**
- Push en vez de polling
- Desacoplamiento entre sistemas
- Eficiente para eventos esporádicos

**Desventajas:**
- Difícil de testear localmente (necesitas ngrok o similar)
- Seguridad: requiere firma/verificación (HMAC)
- Reintentos y idempotencia son tu responsabilidad
- Sin garantía de orden

**Casos de uso:** Integraciones con terceros (Stripe, GitHub, Twilio), eventos entre sistemas

---
## 🔄 Comparativa Rápida

| Tecnología | Formato | Velocidad | Tiempo Real | Complejidad |
|------------|---------|-----------|-------------|-------------|
| SOAP | XML | Lenta | No | Alta |
| REST | JSON/XML | Media | No | Baja |
| GraphQL | JSON | Media | Con subscriptions | Media |
| gRPC | Binario | Alta | Sí (streaming) | Media |
| WebSocket | Cualquiera | Alta | Sí | Media |
| Webhook | JSON | N/A | Push | Baja |

---
## 🎯 Cuándo usar cada uno

> [!tip] Regla general
> - **CRUD simple, APIs públicas** → REST
> - **Frontend con datos complejos** → GraphQL
> - **Microservicios internos, alto rendimiento** → gRPC
> - **Tiempo real bidireccional** → WebSocket
> - **Eventos de terceros** → Webhook
> - **Enterprise legacy, banca** → SOAP

![[1743509489098 1.gif|500]]
