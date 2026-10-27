---
tags:
  - Java
  - Microservicios
  - gRPC
Creado: 2026-08-12
Relacionado:
  - Microservicios
  - Comunicacion Sincrona - Feign y WebClient
---

## 🧠 Qué es gRPC y por qué existe

gRPC es un framework de comunicación RPC (Remote Procedure Call) de Google, sobre HTTP/2 y **Protocol Buffers** (binario) en vez de JSON sobre HTTP/1.1 como REST. Se usa mayormente para comunicación **interna entre microservicios**, no para APIs públicas de cara al cliente (ahí REST sigue siendo más estándar).

```
REST:  Cliente → HTTP/1.1 + JSON     → Servidor   (texto, legible, más pesado)
gRPC:  Cliente → HTTP/2 + Protobuf   → Servidor   (binario, compacto, más rápido)
```

## Por qué es más rápido que REST

- **Protobuf** serializa a binario compacto, mucho más chico que JSON equivalente.
- **HTTP/2** multiplexa varias llamadas sobre una sola conexión TCP (REST/HTTP1.1 abre una conexión por request, o depende de keep-alive).
- El contrato (`.proto`) genera código cliente/servidor fuertemente tipado — sin parseo manual de JSON ni errores de tipeo en nombres de campos.

## Definir el contrato (.proto)

```protobuf
// order.proto
syntax = "proto3";

package com.empresa.order;

option java_multiple_files = true;
option java_package = "com.empresa.order.grpc";

service OrderService {
  rpc GetOrder (GetOrderRequest) returns (OrderResponse);
  rpc CreateOrder (CreateOrderRequest) returns (OrderResponse);
  rpc StreamOrderUpdates (GetOrderRequest) returns (stream OrderStatusUpdate); // streaming
}

message GetOrderRequest {
  int64 order_id = 1;
}

message CreateOrderRequest {
  int64 user_id = 1;
  repeated OrderItem items = 2;
}

message OrderItem {
  string product_id = 1;
  int32 quantity = 2;
}

message OrderResponse {
  int64 order_id = 1;
  string status = 2;
  double total = 3;
}

message OrderStatusUpdate {
  int64 order_id = 1;
  string status = 2;
  int64 timestamp = 3;
}
```

El compilador `protoc` genera automáticamente las clases Java (request/response, stubs de cliente y servidor) a partir de este archivo — no se escriben esas clases a mano.

## Dependencias (Maven, con Spring Boot)

```xml
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-server-spring-boot-starter</artifactId>
    <version>3.1.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-client-spring-boot-starter</artifactId>
    <version>3.1.0.RELEASE</version>
</dependency>
```

```xml
<!-- plugin para generar código Java desde .proto -->
<plugin>
    <groupId>org.xolstice.maven.plugins</groupId>
    <artifactId>protobuf-maven-plugin</artifactId>
    <version>0.6.1</version>
    <configuration>
        <protocArtifact>com.google.protobuf:protoc:3.25.1:exe:${os.detected.classifier}</protocArtifact>
        <pluginId>grpc-java</pluginId>
        <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.62.2:exe:${os.detected.classifier}</pluginArtifact>
    </configuration>
</plugin>
```

## Implementar el servidor

```java
@GrpcService
public class OrderGrpcService extends OrderServiceGrpc.OrderServiceImplBase {

    private final OrderRepository orderRepository;

    public OrderGrpcService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Override
    public void getOrder(GetOrderRequest request, StreamObserver<OrderResponse> responseObserver) {
        Order order = orderRepository.findById(request.getOrderId())
            .orElseThrow(() -> new NoSuchElementException("Order no encontrado"));

        OrderResponse response = OrderResponse.newBuilder()
            .setOrderId(order.getId())
            .setStatus(order.getStatus())
            .setTotal(order.getTotal())
            .build();

        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```

```yaml
grpc:
  server:
    port: 9090   # puerto separado del HTTP normal (ej. 8080)
```

## Consumir desde otro microservicio (cliente)

```java
@Service
public class OrderClientService {

    @GrpcClient("order-service")
    private OrderServiceGrpc.OrderServiceBlockingStub orderStub;

    public OrderResponse buscarOrden(Long orderId) {
        GetOrderRequest request = GetOrderRequest.newBuilder()
            .setOrderId(orderId)
            .build();
        return orderStub.getOrder(request);
    }
}
```

```yaml
grpc:
  client:
    order-service:
      address: 'static://localhost:9090'
      negotiationType: plaintext   # usar TLS en producción
```

## Tipos de comunicación gRPC

| Tipo | Descripción | Caso de uso |
|---|---|---|
| **Unary** | 1 request → 1 response (como REST) | La mayoría de los casos |
| **Server streaming** | 1 request → stream de responses | Notificaciones de progreso, actualizaciones en vivo |
| **Client streaming** | Stream de requests → 1 response | Subir datos en chunks (ej. archivo grande) |
| **Bidirectional streaming** | Stream ↔ Stream | Chat, sincronización en tiempo real |

## gRPC vs REST — cuándo elegir cada uno

| Criterio | REST | gRPC |
|---|---|---|
| Legibilidad/debug | Alta (JSON texto plano) | Baja (binario, necesita herramientas) |
| Performance | Buena | Mejor (binario + HTTP/2) |
| Contrato | OpenAPI (opcional, puede desincronizarse) | `.proto` obligatorio, fuente única de verdad |
| Consumo desde browser | Nativo | Requiere gRPC-Web (capa adicional) |
| Uso típico | APIs públicas, cara al cliente | Comunicación interna microservicio-a-microservicio |
| Streaming | Limitado (SSE, WebSockets aparte) | Nativo, en las 4 variantes |

**Regla práctica**: exponer la API pública/externa en REST (documentable, debuggeable, consumible desde cualquier cliente incluido el navegador), y usar gRPC para las llamadas internas entre microservicios de alto volumen donde la performance y el contrato fuerte importan más que la legibilidad humana directa.

## Errores comunes

- Usar gRPC para APIs públicas de cara a frontend web sin evaluar gRPC-Web — complica el consumo desde JavaScript sin necesidad.
- No versionar el `.proto` con cuidado — cambios incompatibles (renombrar/eliminar un campo con su número) rompen clientes que aún no actualizaron.
- Ignorar que gRPC usa códigos de error propios (`Status.NOT_FOUND`, etc.), distintos a HTTP status codes — mapear mal esto genera manejo de errores inconsistente entre servicios REST y gRPC del mismo sistema.

## Checklist
- [ ] Entiendo por qué gRPC es más eficiente que REST (Protobuf binario + HTTP/2).
- [ ] Sé definir un contrato `.proto` básico con mensajes y un servicio.
- [ ] Sé implementar un servidor y un cliente gRPC con Spring Boot.
- [ ] Distingo los 4 tipos de comunicación (unary, server/client/bidirectional streaming).
- [ ] Tengo criterio para elegir gRPC vs REST según el contexto (interno vs público).

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

