---
tags:
  - Java
  - Spring Boot
  - WebSockets
Creado: 2026-08-12
Relacionado:
  - Spring Boot
---

# WebSockets y Tiempo Real

## Por qué no alcanza con REST para tiempo real

REST es request/response: el cliente siempre inicia. Para que el **servidor** empuje datos al cliente sin que este pregunte (notificaciones, chat, dashboards en vivo), hace falta un canal bidireccional persistente — eso es lo que da WebSocket.

```
REST:      Cliente → pregunta → Servidor → responde → (conexión se cierra)
WebSocket: Cliente ↔ conexión persistente ↔ Servidor   (cualquiera envía en cualquier momento)
```

## Dependencia

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

## STOMP sobre WebSocket (el patrón estándar en Spring)

Spring no expone WebSocket "crudo" como primera opción — usa **STOMP** (Simple Text Oriented Messaging Protocol) encima, que agrega un modelo de pub/sub con destinos (`/topic/...`), similar en espíritu a cómo se usa un message broker.

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
            .setAllowedOrigins("http://localhost:3000")
            .withSockJS();   // fallback para navegadores sin soporte WebSocket nativo
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic", "/queue");   // broker en memoria (simple)
        registry.setApplicationDestinationPrefixes("/app"); // prefijo para mensajes del cliente al servidor
        registry.setUserDestinationPrefix("/user");         // mensajes dirigidos a un usuario específico
    }
}
```

## Controller que recibe y envía mensajes (broadcast)

```java
@Controller
public class ChatController {

    // Cliente envía a /app/chat.enviar → este método procesa
    @MessageMapping("/chat.enviar")
    @SendTo("/topic/mensajes")   // reenvía a todos los suscritos a /topic/mensajes
    public MensajeDTO enviarMensaje(MensajeDTO mensaje) {
        mensaje.setTimestamp(Instant.now());
        return mensaje;   // se transmite tal cual a todos los suscriptores
    }
}
```

## Enviar mensajes desde cualquier parte del código (no solo desde un handler)

```java
@Service
@RequiredArgsConstructor
public class NotificacionService {

    private final SimpMessagingTemplate messagingTemplate;

    // Broadcast a todos los suscritos
    public void notificarActualizacionPedido(PedidoDTO pedido) {
        messagingTemplate.convertAndSend("/topic/pedidos", pedido);
    }

    // A un usuario específico (requiere autenticación WebSocket configurada)
    public void notificarUsuario(String username, NotificacionDTO notificacion) {
        messagingTemplate.convertAndSendToUser(username, "/queue/notificaciones", notificacion);
    }
}
```

## Cliente JavaScript (referencia, para entender el flujo completo)

```javascript
const socket = new SockJS('/ws');
const stompClient = Stomp.over(socket);

stompClient.connect({}, () => {
    // Suscribirse a un topic
    stompClient.subscribe('/topic/mensajes', (mensaje) => {
        console.log(JSON.parse(mensaje.body));
    });

    // Enviar un mensaje al servidor
    stompClient.send('/app/chat.enviar', {}, JSON.stringify({ texto: 'Hola' }));
});
```

## Autenticación en WebSockets

```java
@Configuration
public class WebSocketSecurityConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                StompHeaderAccessor accessor = MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
                if (StompCommand.CONNECT.equals(accessor.getCommand())) {
                    String token = accessor.getFirstNativeHeader("Authorization");
                    // validar JWT y setear el Principal del accessor
                    Authentication auth = jwtService.validarYObtenerAuth(token);
                    accessor.setUser(auth);
                }
                return message;
            }
        });
    }
}
```

La conexión WebSocket se autentica **una vez** al conectar (handshake), no en cada mensaje individual como en REST con cada request.

## Broker externo (RabbitMQ/ActiveMQ) en vez del broker simple en memoria

```java
@Override
public void configureMessageBroker(MessageBrokerRegistry registry) {
    registry.enableStompBrokerRelay("/topic", "/queue")
        .setRelayHost("localhost")
        .setRelayPort(61613)
        .setClientLogin("guest")
        .setClientPasscode("guest");
}
```

El broker simple en memoria de Spring **no escala horizontalmente** — si hay 2 instancias de la app, un cliente conectado a la instancia A no recibe mensajes originados en la instancia B. Un broker externo (RabbitMQ con STOMP plugin) resuelve esto para producción con múltiples réplicas.

## WebSockets vs Server-Sent Events (SSE) vs Kafka/RabbitMQ

| Tecnología | Dirección | Cuándo usar |
|---|---|---|
| **WebSocket** | Bidireccional | Chat, colaboración en tiempo real, juegos |
| **SSE** (`Flux` en WebFlux) | Solo servidor → cliente | Notificaciones simples, feeds, progreso de una tarea |
| **Kafka/RabbitMQ** | Servicio a servicio (backend) | Comunicación interna entre microservicios, no directo al navegador |

Para "solo necesito avisarle al cliente cuando algo cambió" (sin que el cliente responda por el mismo canal), SSE es más simple que WebSocket — menos infraestructura, funciona sobre HTTP normal.

## Errores comunes

- No planear para múltiples instancias — el broker simple en memoria se rompe apenas hay más de un Pod/instancia detrás de un load balancer (relevante en Kubernetes con `replicas > 1`).
- Autenticar solo el HTTP handshake inicial y no validar después — igual sigue siendo válido porque la conexión persiste, pero hay que revocar la conexión si el token expira o el usuario pierde permisos.
- Usar WebSocket para casos donde SSE o incluso polling simple hubiera bastado, agregando complejidad innecesaria.
- No manejar reconexión del lado del cliente — las conexiones WebSocket se caen (red, reinicio de Pod) y la app debe reconectar automáticamente, no asumir que el canal es eterno.

## Checklist
- [ ] Entiendo por qué WebSocket resuelve algo que REST no puede (push del servidor).
- [ ] Sé configurar STOMP sobre WebSocket con destinos `/topic` y `/app`.
- [ ] Sé enviar mensajes broadcast y a un usuario específico.
- [ ] Sé por qué el broker simple en memoria no sirve con múltiples instancias, y qué lo reemplaza.
- [ ] Tengo criterio para elegir entre WebSocket, SSE y mensajería backend según el caso.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

