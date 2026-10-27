---
tags:
  - Java
  - Cloud
  - Red
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - 04 - API Gateway
Palabras clave:
  - VNet, VPC, subred privada, NAT, Private Link, WAF, ALB, APIM, X-Forwarded-For, TLS
---

## 🧠 Concepto

La red define **qué puede hablar con qué**. Es la capa donde se contiene un compromiso: si un servicio es vulnerado, la topología decide si el atacante llega a la base de datos o se queda encerrado.

## 🏗️ La topología base

```
Internet
   │
   ▼
┌─────────────────────────────────────────────┐
│  Subred pública                              │
│  ├── Balanceador / WAF                       │
│  └── NAT (salida a internet de lo privado)   │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  Subred privada — aplicación                 │
│  └── corebank-api  (sin IP pública)          │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  Subred privada — datos                      │
│  └── PostgreSQL  (sin acceso desde internet) │
└─────────────────────────────────────────────┘
```

| Concepto | Azure | AWS |
|---|---|---|
| Red virtual | VNet | VPC |
| Segmento | Subnet | Subnet |
| Firewall de instancia | NSG | Security Group |
| Firewall de subred | NSG | NACL |
| Salida a internet | NAT Gateway | NAT Gateway |
| Acceso privado a servicios | Private Endpoint | VPC Endpoint / PrivateLink |

> [!danger] Nada que no reciba tráfico de internet debe tener IP pública
> La aplicación se expone **a través del balanceador**; la base no se expone en absoluto. Un endpoint público de base de datos es una invitación abierta, aunque tenga contraseña.

> [!warning] Security Group y NACL no son lo mismo
> El *Security Group* de AWS es **con estado**: si permitís la entrada, la respuesta sale sola. La NACL es **sin estado**: hay que permitir explícitamente ida y vuelta. Es una fuente clásica de "funciona en un lado y no en el otro".
> El NSG de Azure es con estado, como el Security Group.

## 🚪 Exponer la aplicación

| Nivel | Azure | AWS | Para qué |
|---|---|---|---|
| Global (CDN + WAF) | **Front Door** | **CloudFront** | latencia mundial, DDoS |
| L7 regional | **Application Gateway** | **ALB** | enrutado por ruta, TLS, WAF |
| L4 | Load Balancer | NLB | TCP puro, altísimo rendimiento |
| Gestión de APIs | **APIM** | **API Gateway** | cuotas, claves, versionado, transformación |

```
Usuario → Front Door / CloudFront   (caché, WAF, terminación TLS cercana)
        → App Gateway / ALB          (enrutado, health checks)
        → APIM / API Gateway         (autenticación, cuotas, versionado)
        → corebank-api
```

> [!tip] No hace falta tener todas las capas
> Para un backend interno, un balanceador L7 alcanza. La puerta de enlace de APIs se justifica cuando exponés a terceros y necesitás cuotas por cliente, claves, versionado de contratos y monetización — como en [[14 - Open Banking]].
> Cada capa suma latencia y costo: agregá la que resuelva un problema concreto.

## 🔐 Acceso privado a servicios gestionados

Por defecto, hablar con la base o con el almacén de secretos sale a internet — aunque los dos extremos sean tuyos.

```
❌ App (subred privada) → NAT → internet → endpoint público de la base
✅ App (subred privada) → Private Endpoint → base   (nunca sale de la red)
```

```bash
# Azure: endpoint privado para PostgreSQL
az network private-endpoint create \
  --name pe-postgres --vnet-name vnet-corebank --subnet snet-datos \
  --private-connection-resource-id $ID_SERVIDOR \
  --group-id postgresqlServer --connection-name conn-postgres
```

| Ventaja | Detalle |
|---|---|
| Seguridad | el tráfico nunca toca internet |
| Cumplimiento | requisito habitual en banca |
| Latencia | ligeramente menor |
| Costo | **evita cargos de NAT y de egreso** |

> [!tip] El endpoint privado suele pagarse solo
> El NAT Gateway cobra por hora **y por gigabyte procesado**. Una aplicación que consulta mucho la base o el almacenamiento a través del NAT puede generar un costo sorprendente. Con endpoints privados ese tráfico no pasa por el NAT.
> Es de las optimizaciones de costo que además mejoran la seguridad — ver [[13 - Costos y FinOps]].

## 🛡️ WAF

Filtra ataques conocidos antes de que lleguen a la aplicación: inyección SQL, XSS, exploits de la lista OWASP.

```bash
# Empezar en modo detección, no en bloqueo
az network application-gateway waf-config set --enabled true \
  --firewall-mode Detection --rule-set-version 3.2
```

> [!warning] Un WAF en bloqueo desde el día uno rompe cosas
> Las reglas genéricas producen falsos positivos: un JSON con comillas o un campo de texto libre pueden parecerse a un ataque. Se arranca en **detección**, se revisan las coincidencias unos días, se ajustan las excepciones y recién ahí se pasa a bloqueo.

> [!danger] El WAF no reemplaza el código seguro
> Es una capa adicional, no una solución. Consultas parametrizadas, validación de entrada y codificación de salida siguen siendo obligatorias — ver [[07 - OWASP Top 10 en Java]].

## 🔒 TLS

```
Usuario ──TLS──► Balanceador ──?──► Aplicación
```

| Modelo | Del balanceador a la app | Cuándo |
|---|---|---|
| Terminación | HTTP plano | red privada de confianza |
| Reencriptado | HTTPS | datos sensibles, cumplimiento |
| Paso a través | TLS extremo a extremo | la app maneja el certificado |

> [!warning] Para datos financieros, cifrá también el tramo interno
> "Es la red privada" no alcanza en un modelo de confianza cero: un servicio comprometido dentro de la red puede leer tráfico en claro. PCI-DSS exige cifrado en tránsito, y eso no distingue tramos internos de externos.

```java
// La app tiene que saber que está detrás de un proxy
```

```yaml
server:
  forward-headers-strategy: framework      # interpreta X-Forwarded-*
```

> [!danger] Sin esto, la app cree que todo viene de la IP del balanceador
> Los logs registran la IP del proxy en vez de la del usuario, el rate limiting por IP limita al balanceador entero, y las URLs generadas salen con `http://` y el host interno.
> El síntoma clásico: los correos de recuperación de contraseña llevan un enlace a `http://10.0.1.5:8080/...`.

```java
// Obtener la IP real con cuidado
public String ipReal(HttpServletRequest req) {
    String forwarded = req.getHeader("X-Forwarded-For");
    // El cliente puede FALSIFICAR esta cabecera: solo confiá si viene de tu proxy
    return forwarded != null && esProxyDeConfianza(req.getRemoteAddr())
        ? forwarded.split(",")[0].trim()
        : req.getRemoteAddr();
}
```

## ⏱️ Timeouts en cascada

```
Cliente             30 s
  CDN               25 s
    Balanceador     20 s
      Aplicación    15 s
        Base         5 s
```

> [!danger] Cada capa debe rendirse **antes** que la de afuera
> Si el timeout de la base fuera mayor que el del balanceador, el balanceador corta la conexión y la consulta sigue corriendo: consumís recursos por un resultado que nadie va a leer. Con carga, eso satura el pool.

| Balanceador | Timeout de inactividad por defecto |
|---|---|
| AWS ALB | 60 s |
| Azure Application Gateway | ~20 s (configurable) |
| Azure Front Door | ~60 s |

> [!warning] El timeout del balanceador limita las peticiones largas
> Una exportación que tarda dos minutos falla contra un ALB con 60 s de inactividad. La solución no es subir el timeout: es **convertirla en asíncrona** — devolver `202` con un identificador y que el cliente consulte el resultado.

## 🩺 Health checks

```yaml
management:
  endpoint:
    health:
      probes:
        enabled: true
  endpoints:
    web:
      exposure:
        include: health          # el balanceador solo necesita esto
```

| Sonda | Endpoint | Efecto |
|---|---|---|
| Del balanceador | `/actuator/health/readiness` | saca la instancia de rotación |
| Del orquestador (liveness) | `/actuator/health/liveness` | reinicia el contenedor |

> [!danger] Nunca expongas todos los endpoints de Actuator
> `/actuator/env`, `/actuator/configprops` y `/actuator/heapdump` revelan configuración y estado interno. Si el balanceador enruta `/actuator/**` al servicio, quedan accesibles desde internet.

## 🔄 Sesiones pegajosas

```
❌ Sticky sessions: el balanceador manda al usuario siempre a la misma instancia
```

> [!warning] Las sesiones pegajosas son un parche para un diseño con estado
> Rompen el balanceo (una instancia se sobrecarga), rompen el escalado (las nuevas no reciben usuarios existentes) y rompen el despliegue (al reemplazar una instancia sus usuarios pierden la sesión).
> La solución es no tener estado: identidad en el token, estado compartido en Redis o en la base. Ver [[01 - Modelo Mental Cloud]].

## 🌍 Distribución global

```
Usuario en Lima      → punto de presencia en Lima      → origen en São Paulo
Usuario en Madrid    → punto de presencia en Madrid    → origen en Irlanda
```

| Servicio | Qué aporta |
|---|---|
| CDN | caché de estáticos cerca del usuario |
| Terminación TLS en el borde | menos latencia de handshake |
| Enrutado por cercanía | va al origen más cercano |
| Protección DDoS | absorbe el ataque antes del origen |

> [!tip] Terminar TLS cerca del usuario reduce mucho la latencia percibida
> El handshake TLS son varias idas y vueltas. Hacerlas contra un punto a 20 ms en vez de a 200 ms mejora el tiempo de la primera respuesta aunque el contenido venga igual del origen.

## 🧪 Verificaciones

```java
@Test
void la_app_registra_la_ip_real_del_cliente() {
    api.conCabecera("X-Forwarded-For", "203.0.113.45").get("/api/v1/cuentas");
    assertEquals("203.0.113.45", ultimoLog().get("ip"));
}

@Test
void una_cabecera_forwarded_de_origen_no_confiable_se_ignora() { ... }

@Test
void los_endpoints_sensibles_de_actuator_no_estan_expuestos() {
    assertEquals(404, api.get("/actuator/env").statusCode());
    assertEquals(404, api.get("/actuator/heapdump").statusCode());
}

@Test
void las_urls_generadas_usan_el_host_publico_y_https() {
    var respuesta = api.conCabeceras(Map.of(
        "X-Forwarded-Proto", "https", "X-Forwarded-Host", "api.corebank.com")).post("/cuentas");
    assertTrue(respuesta.header("Location").startsWith("https://api.corebank.com"));
}
```

```bash
# ¿La base es alcanzable desde internet?
nc -zv mi-servidor.postgres.database.azure.com 5432    # debe fallar ✅

# ¿La app tiene IP pública?
az containerapp show --name corebank --query properties.configuration.ingress.external
```

## ⚠️ Errores Comunes

> [!warning] Los que exponen o rompen el servicio
> - **Base de datos con endpoint público.**
> - **Aplicación con IP pública** en vez de detrás del balanceador.
> - **Sin `forward-headers-strategy`**: IPs y URLs incorrectas.
> - **Confiar en `X-Forwarded-For`** sin verificar que venga del proxy.
> - **Timeouts crecientes hacia adentro.**
> - **WAF en bloqueo desde el primer día.**
> - **Exponer todo Actuator** a través del balanceador.
> - **Sesiones pegajosas** en vez de resolver el estado.
> - **Todo el tráfico saliendo por NAT** pudiendo usar endpoints privados.

## 📌 Notas Rápidas
- La regla que ordena la topología: **público solo lo que recibe tráfico de internet**. Todo lo demás en subredes privadas, y el acceso a servicios gestionados por endpoint privado.
- `forward-headers-strategy: framework` es una línea de configuración que resuelve una familia entera de bugs raros (IPs mal registradas, enlaces con host interno, redirecciones a HTTP). Vale ponerla siempre que haya un proxy delante.
- Los endpoints privados mejoran seguridad y costo al mismo tiempo, que es una combinación poco frecuente. Suele ser de lo primero que revisaría en una arquitectura existente.
- Se aplica en [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[09 - Observabilidad Cloud]]
- **Siguiente**: [[11 - Serverless con Java]]
- **Se conecta con**: [[04 - API Gateway]], [[04 - APIM y Application Gateway]], [[05 - CORS y CSRF]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

