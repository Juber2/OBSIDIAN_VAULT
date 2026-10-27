---
tags:
  - Java
  - Cloud
  - Azure
  - AWS
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - DevOps con Java
Palabras clave:
  - region, zona de disponibilidad, SLA, responsabilidad compartida, efimero, cuota
---

## 🧠 Concepto

La nube no es "un servidor alquilado". Lo que comprás es el **servicio gestionado**: parches, alta disponibilidad, escalado y respaldos operados por otro. Si la usás como un servidor, pagás precio de nube y obtenés servicio de servidor.

Esta nota fija el vocabulario y las restricciones que aplican a **todas** las demás del módulo.

## 🌍 Regiones y zonas

| Concepto | Qué es | Fallo que tolera |
|---|---|---|
| **Región** | un área geográfica (Brasil Sur, us-east-1) | desastre regional |
| **Zona de disponibilidad** | datacenters independientes dentro de una región | corte eléctrico, incendio, red |
| **Punto de presencia (edge)** | nodo de CDN cerca del usuario | latencia |

```
Región: Brasil Sur
├── Zona 1  ← energía, refrigeración y red independientes
├── Zona 2
└── Zona 3
```

> [!danger] Desplegar en una sola zona es no tener alta disponibilidad
> Es el error de despliegue más común y el más caro: tres réplicas en la misma zona no toleran nada. La zona cae y se van las tres juntas.
> Distribuir entre zonas suele ser gratis o casi (dentro de la región), y es la diferencia entre un SLA con y sin garantía.

> [!warning] El tráfico entre zonas se factura
> Es barato por gigabyte pero constante. Un sistema con servicios que se hablan mucho y están repartidos entre zonas puede gastar más en tráfico que en cómputo. Es un costo invisible hasta que llega la factura — ver [[13 - Costos y FinOps]].

| Necesito tolerar | Despliego en |
|---|---|
| Fallo de una máquina | varias instancias |
| Fallo de una zona | **varias zonas** ← el mínimo razonable |
| Fallo de una región | varias regiones ([[14 - Multiregion y DR]]) |

## 📜 Responsabilidad compartida

El proveedor opera parte del sistema, vos operás el resto. Dónde está la línea depende del modelo:

| Capa | IaaS | Contenedores gestionados | PaaS | Serverless |
|---|---|---|---|---|
| Código de la app | 🟦 vos | 🟦 vos | 🟦 vos | 🟦 vos |
| Dependencias | 🟦 vos | 🟦 vos | 🟦 vos | 🟦 vos |
| Runtime (JVM) | 🟦 vos | 🟦 vos | 🟩 nube | 🟩 nube |
| Sistema operativo | 🟦 vos | 🟩 nube | 🟩 nube | 🟩 nube |
| Virtualización | 🟩 nube | 🟩 nube | 🟩 nube | 🟩 nube |
| Hardware y red física | 🟩 nube | 🟩 nube | 🟩 nube | 🟩 nube |

> [!danger] La seguridad de tus datos y accesos es siempre tuya
> Ningún modelo transfiere esa responsabilidad. Un bucket de almacenamiento expuesto públicamente, un secreto en el repositorio o un permiso demasiado amplio son responsabilidad del cliente en los cuatro modelos.
> La nube te da herramientas de seguridad, no seguridad.

## ⚡ Todo es efímero

La suposición central que cambia respecto de un servidor propio: **la instancia donde corre tu app puede desaparecer en cualquier momento**, y eso es normal — no un fallo.

Consecuencias directas sobre el código:

```java
// ❌ Estado en disco local: se pierde con la instancia
Files.write(Path.of("/app/uploads/" + nombre), bytes);

// ✅ Almacenamiento de objetos
blobClient.upload(nombre, bytes);          // ver [[08 - Almacenamiento de Objetos]]
```

```java
// ❌ Sesión en memoria: el usuario "pierde la sesión" al escalar
httpSession.setAttribute("carrito", carrito);

// ✅ Sin estado: token o almacén compartido
// el JWT lleva la identidad; el carrito vive en Redis o en la base
```

```java
// ❌ Job programado en cada instancia: se ejecuta N veces
@Scheduled(cron = "0 0 3 * * *")
public void cierre() { ... }

// ✅ Coordinado: uno solo lo ejecuta
@Scheduled(cron = "0 0 3 * * *")
@SchedulerLock(name = "cierre", lockAtMostFor = "PT2H")
public void cierre() { ... }
```

> [!warning] La app tiene que arrancar rápido y apagar prolijo
> El orquestador reemplaza instancias constantemente: al escalar, al desplegar, al reciclar nodos. Una app que tarda tres minutos en arrancar hace que cada despliegue sea lento y cada pico de carga llegue tarde.
> Y al recibir `SIGTERM` hay que **dejar de aceptar peticiones nuevas, terminar las en curso y cerrar recursos** — el apagado ordenado no es opcional.

```yaml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

## 📊 SLA — qué promete el proveedor

| SLA | Caída máxima al mes | Uso típico |
|---|---|---|
| 99% | 7,2 h | sin garantía real |
| 99,9% ("tres nueves") | 43 min | servicio estándar |
| 99,95% | 21 min | con redundancia de zona |
| 99,99% ("cuatro nueves") | 4,3 min | multi-zona con réplicas |
| 99,999% | 26 s | multi-región |

> [!danger] Los SLA se multiplican, no se promedian
> Si tu app depende de cómputo (99,95%), base (99,99%) y almacenamiento (99,9%), tu disponibilidad máxima teórica es:
> ```
> 0,9995 × 0,9999 × 0,999 = 0,9984  →  99,84%  ≈ 70 min/mes
> ```
> **Cada dependencia baja el techo.** Un sistema con quince servicios gestionados en serie no puede prometer cuatro nueves, por mejor que sea cada pieza.

La forma de subir el techo es tener **caminos alternativos**: caché que responde si la base no está, degradación funcional, reintentos. Ver [[08 - Alta Disponibilidad]].

> [!tip] El SLA es un crédito en la factura, no una garantía
> Si el proveedor incumple, te devuelve un porcentaje del costo del servicio. No te compensa el negocio perdido. El SLA sirve para dimensionar la arquitectura, no para transferir el riesgo.

## 🚧 Cuotas y límites

Todo servicio gestionado tiene límites, y algunos aparecen en el peor momento.

| Tipo | Ejemplo | Qué pasa al llegar |
|---|---|---|
| **Cuota ampliable** | número de instancias, IPs | se pide aumento (tarda) |
| **Límite duro** | tamaño de mensaje, timeout máximo | no se puede cambiar |
| **Límite de tasa** | peticiones por segundo a la API | se rechaza con `429` |
| **Límite de conexiones** | conexiones a la base | se rechaza la conexión |

> [!danger] El límite de conexiones a la base es el que más sorprende
> Una base gestionada pequeña puede admitir 100 conexiones. Con 20 instancias y un pool de 10 cada una, pedís 200 y la mitad falla.
> ```
> conexiones_totales = instancias × tamaño_del_pool
> ```
> Es una multiplicación que hay que hacer **antes** de escalar horizontalmente, no después — ver [[06 - Bases de Datos Gestionadas]].

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10       # × número de instancias ≤ límite de la base
      minimum-idle: 2
      connection-timeout: 3000
      max-lifetime: 1800000       # menor que el timeout del lado servidor
```

## 🔁 Reintentos con servicios gestionados

Los servicios gestionados aplican **limitación de tasa** (*throttling*): ante exceso de carga responden `429` o `503` en vez de degradarse. Es un comportamiento esperado, no un fallo.

```java
@Bean
public Retry politicaCloud() {
    return Retry.of("servicioGestionado", RetryConfig.custom()
        .maxAttempts(4)
        .intervalFunction(IntervalFunction.ofExponentialRandomBackoff(
            Duration.ofMillis(500), 2.0, 0.5))       // jitter obligatorio
        .retryOnResult(r -> esLimitacionDeTasa(r))
        .build());
}
```

> [!warning] Respetá el `Retry-After` cuando venga
> Muchos servicios indican en la respuesta cuánto esperar. Ignorarlo y reintentar de inmediato empeora la limitación y puede escalarla a un bloqueo temporal.

## 🎯 Elegir el modelo de servicio

```
¿Puedo usar un servicio gestionado en vez de operarlo yo?
├── SÍ  → usalo, salvo que el costo o una restricción lo impidan
└── NO  → ¿por qué no? (costo · cumplimiento · una función que falta)
```

| Prefiero | Antes que |
|---|---|
| Base gestionada | Postgres en una VM |
| Cola gestionada | RabbitMQ propio |
| Almacén de secretos | archivos de configuración |
| Balanceador gestionado | nginx propio |
| Identidad gestionada | usuarios y contraseñas |

> [!tip] La pregunta correcta no es "¿cuánto cuesta el servicio?"
> Es **"¿cuánto cuesta operarlo yo?"** — parches, respaldos, monitoreo, alta disponibilidad, guardias, y el tiempo del equipo. Una base gestionada casi siempre sale más barata que una persona manteniendo la propia, y falla mucho menos.
> La excepción real es el volumen muy alto y sostenido, donde el sobreprecio del servicio gestionado supera el costo de operarlo.

## ⚠️ Errores Comunes

> [!warning] Los que se pagan pronto
> - **Desplegar en una sola zona** y creer que hay alta disponibilidad.
> - **Estado en disco local**: se pierde con cada reemplazo de instancia.
> - **Sesión en memoria**: se rompe al escalar.
> - **Jobs programados sin coordinación**: se ejecutan N veces.
> - **No calcular `instancias × pool`** contra el límite de conexiones.
> - **Sin apagado ordenado**: peticiones cortadas en cada despliegue.
> - **Prometer un SLA mayor** que el producto de las dependencias.
> - **Reintentar sin jitter ni `Retry-After`**: agrava la limitación de tasa.
> - **Suponer que la nube resuelve la seguridad**: los accesos y los datos son tuyos.

## 📌 Notas Rápidas
- Si te llevás una sola idea: **todo es efímero**. Casi todas las buenas prácticas de este módulo se derivan de asumir que la instancia donde corre tu código va a desaparecer sin aviso.
- La multiplicación de SLA es un buen argumento contra sumar dependencias sin pensar: cada servicio gestionado que agregás en el camino crítico baja tu techo de disponibilidad.
- Las cuotas se piden con anticipación. Descubrir el día del lanzamiento que necesitás una ampliación que tarda 48 h es un clásico evitable.

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Siguiente**: [[02 - Computo para Apps Java]]
- **Se conecta con**: [[14 - Multiregion y DR]], [[01 - Falacias de la Computacion Distribuida]], [[Azure]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

