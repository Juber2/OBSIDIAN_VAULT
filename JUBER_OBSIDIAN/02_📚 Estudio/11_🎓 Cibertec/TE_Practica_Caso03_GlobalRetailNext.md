---
tags:
  - Cibertec
  - TecnologiasEmergentes
  - Practica
  - Caso03
Actualizado: 2026-06-24
Curso: "[[Tecnologias_Emergentes]]"
---

# Práctica — Caso 03: GLOBAL RETAIL NEXT

> Multinacional retail con 4,000+ tiendas en América, Europa y Asia. Busca transformar la experiencia de compra con IA Generativa, XR, Blockchain, Edge Computing, Gemelos Digitales y Cómputo Cuántico.

---

## P1. Madurez Tecnológica de Todas las Iniciativas

| Tecnología | Hype Cycle | Tech Radar | Curva S | Aplicación en Retail |
|---|---|---|---|---|
| **IA Generativa (atención personalizada)** | Pico → entrando al Abismo | Trial | Crecimiento acelerado | Chatbots, recomendación de productos, generación de descripciones |
| **RAG (asistente empleados/clientes)** | Rampa de consolidación | Trial | Crecimiento acelerado | Consultas sobre inventario, políticas, productos |
| **Edge Computing (tiendas inteligentes)** | Rampa de consolidación | Adoptar | Crecimiento acelerado | Análisis de comportamiento local, inventario en tiempo real |
| **IoT (sensores, cámaras)** | Meseta de productividad | Adoptar | Madurez | Ya estándar en retail avanzado |
| **XR (experiencias de compra)** | Rampa de consolidación | Trial | Crecimiento | Probadores virtuales, showrooms AR |
| **Blockchain (trazabilidad)** | Abismo de desilusión | Evaluar | Incubación | Registro de cadena de suministro |
| **Gemelos Digitales (supply chain)** | Pico de expectativas | Trial | Inicio crecimiento | Simulación de cadena de suministro |
| **Cómputo Cuántico (logística)** | Pico de expectativas | Evaluar/Pausar | Incubación | Optimización de rutas y distribución |

### Análisis diferenciado por iniciativa

**IA Generativa + RAG:** es la más madura para retail. Amazon, Zara y H&M ya tienen asistentes de compra con IA en producción. El riesgo principal es la alucinación en recomendaciones de productos con precios incorrectos.

**Edge Computing + IoT:** completamente maduro. Amazon Go lleva desde 2018 con tiendas sin cajero basadas en edge + visión por computadora. Walmart, Carrefour y Tesco ya lo usan a escala.

**XR:** probadores virtuales (AR) ya funcionan en IKEA, Sephora, Warby Parker con datos de conversión positivos. Showrooms VR aún en piloto en retail de lujo.

**Blockchain en supply chain:** casos reales existen (Walmart + IBM Food Trust para trazabilidad alimentaria, LVMH + Aura para lujo), pero la adopción masiva es lenta por complejidad de integración con proveedores.

**Gemelos Digitales supply chain:** Unilever, Procter & Gamble y Amazon los usan para sus cadenas. Para 4,000 tiendas es viable pero complejo.

**Cómputo Cuántico:** D-Wave ya vende acceso cloud a algoritmos de optimización de rutas. Resultados prometedores en instancias pequeñas pero ventaja cuántica no demostrada a escala real en logística.

---

## P2. Arquitectura Omnicanal Cloud Native

```
┌──────────────────────────────────────────────────────────────────────────┐
│                     CANALES DE CLIENTE (Omnicanal)                       │
│  Tienda física │ App móvil │ Web e-commerce │ Kiosco │ Redes sociales    │
│  Asistente voz │ Visor AR  │ Smart TV       │ Chatbot WhatsApp           │
└──────────────────────────────────┬───────────────────────────────────────┘
                                   │ Todos los canales → misma experiencia
┌──────────────────────────────────▼───────────────────────────────────────┐
│               API GATEWAY + CDN (CloudFront / Akamai)                   │
│         Rate limiting │ Auth (OAuth2 + PKCE) │ A/B Testing               │
└──────────┬───────────────────┬───────────────────┬────────────────────────┘
           │                   │                   │
┌──────────▼──────────┐ ┌──────▼────────┐ ┌───────▼──────────────────────┐
│  MOTOR DE           │ │  PLATAFORMA   │ │  GESTIÓN DE PEDIDOS          │
│  PERSONALIZACIÓN    │ │  COMERCIO     │ │  Y SUPPLY CHAIN              │
│  (IA + RAG)         │ │               │ │                              │
│  Recomendación      │ │  Catálogo     │ │  OMS (Order Management)      │
│  personalizada      │ │  unificado    │ │  WMS (Warehouse Mgmt)        │
│  Asistente RAG      │ │  Precios      │ │  TMS (Transport Mgmt)        │
│  empleados          │ │  dinámicos    │ │  Gemelo Digital              │
│  Asistente RAG      │ │  Carrito      │ │  supply chain                │
│  clientes           │ │  unificado    │ │                              │
└──────────┬──────────┘ └───────────────┘ └──────────────────────────────┘
           │
┌──────────▼──────────────────────────────────────────────────────────────┐
│                    PLATAFORMA CLOUD NATIVE (Multi-región)                │
│  Región América │ Región Europa (GDPR) │ Región Asia                    │
│                                                                          │
│  Apache Kafka (eventos unificados)    │  Kubernetes + Istio             │
│  Data Lake (Delta Lake / Iceberg)     │  MLOps: Kubeflow + MLflow       │
│  Microservicios de dominio (DDD)      │  Feature Store (Feast)          │
│  Event Sourcing + CQRS                │  Observabilidad: Grafana + Otel │
│  GraphQL Federation (API unificada)   │  FinOps: spot + reserved        │
└──────────┬──────────────────────────────────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────────────────────────────────┐
│                    CAPA BLOCKCHAIN                                       │
│  Hyperledger Fabric / Ethereum privado                                  │
│  Trazabilidad de productos (farm-to-shelf)                              │
│  Certificados de autenticidad (productos de lujo)                       │
│  Smart contracts con proveedores                                        │
└──────────┬──────────────────────────────────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────────────────────────────────┐
│               EDGE COMPUTING (por tienda / zona geográfica)             │
│  Servidor edge por tienda (NVIDIA Jetson / Intel NUC)                   │
│  Funciona offline: inventario local, recomendaciones cacheadas          │
│  Procesa cámaras y sensores localmente sin enviar video a cloud         │
└──────────┬──────────────────────────────────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────────────────────────────────┐
│            DISPOSITIVOS IOT + XR EN TIENDA                              │
│  Cámaras inteligentes │ Sensores de estantes │ Etiquetas RFID           │
│  Beacons BLE          │ Espejos inteligentes │ Visores AR/MR            │
│  Pantallas digitales  │ POS inteligente      │ Sensores de aforo        │
└─────────────────────────────────────────────────────────────────────────┘
```

### Principios de diseño omnicanal

| Principio | Implementación |
|---|---|
| **Single source of truth** | Catálogo, inventario y perfil del cliente unificados en una sola plataforma |
| **Consistencia de experiencia** | El cliente que agrega al carrito en la app puede pagar en tienda física y devolver online |
| **Datos en tiempo real** | Kafka garantiza que el inventario se actualiza en todos los canales en < 500ms |
| **Personalización cross-canal** | El historial de compras en tienda alimenta las recomendaciones de la app y viceversa |
| **Resilencia local** | Si cae el cloud, la tienda física sigue operando con el edge local |

---

## P3. Edge Computing en Tiendas Inteligentes

### Funciones del nodo edge por tienda

```
TIENDA FÍSICA
│
├── CÁMARAS INTELIGENTES (procesadas localmente en edge)
│   ├── Conteo de aforo en tiempo real (sin identificar personas — privacidad)
│   ├── Análisis de zonas calientes: qué áreas tienen más tráfico
│   ├── Detección de productos fuera de lugar en estantes
│   └── Alertas de seguridad: conducta sospechosa sin reconocimiento facial
│
├── SENSORES DE ESTANTES (IoT)
│   ├── Peso: detecta cuándo un producto se agota
│   ├── RFID: inventario en tiempo real sin escaneo manual
│   └── Temperatura: cadena de frío en alimentos y farmacia
│
├── MOTOR DE RECOMENDACIÓN LOCAL (modelo ML cacheado)
│   ├── Pantallas personalizadas según perfil del cliente (app abierta)
│   ├── Sugerencias en kiosco sin necesidad de consultar cloud
│   └── Precios dinámicos aprobados por cloud y aplicados localmente
│
└── SINCRONIZACIÓN CON CLOUD (cuando hay conectividad)
    ├── Datos de comportamiento agregados (no video crudo)
    ├── Actualización de modelos ML y reglas de negocio
    └── Sincronización de inventario y precios
```

### Ventajas del edge en retail

| Ventaja | Impacto en tienda |
|---|---|
| **Latencia < 50ms** | Actualización de precio en la etiqueta digital antes de que el cliente llegue a caja |
| **Privacidad por diseño** | El video de las cámaras nunca sale de la tienda — se procesa localmente y se descarta |
| **Operación offline** | Si cae internet, la tienda sigue vendiendo con inventario local sincronizado |
| **Reducción de costos cloud** | Solo se envían datos agregados (no streaming de video = 95% menos tráfico) |
| **Cumplimiento GDPR** | Datos de clientes europeos no salen de la UE — se procesan en el edge local |

### Arquitectura edge por tienda

```
Servidor Edge (por tienda)
│  CPU: Intel Xeon / AMD EPYC
│  GPU: NVIDIA RTX 4000 (visión por computadora)
│  RAM: 64GB │ SSD: 2TB NVMe
│  OS: Ubuntu Server + K3s (Kubernetes ligero)
│
├── Container: vision-ai (análisis de cámaras)
├── Container: inventory-engine (inventario RFID + sensores)
├── Container: recommendation-cache (modelo ML local)
├── Container: pos-gateway (integración caja registradora)
└── Container: sync-agent (sincronización con cloud)
```

---

## P4. Solución RAG para Empleados y Clientes

### Dos asistentes con bases de conocimiento distintas

```
┌─────────────────────────────────────────────────────────────┐
│               ASISTENTE PARA CLIENTES                       │
│                                                             │
│  Canal: App móvil, web, kiosco en tienda, WhatsApp         │
│                                                             │
│  Knowledge Base:                                            │
│  - Catálogo completo de productos con especificaciones      │
│  - Política de devoluciones por país                        │
│  - Estado de pedidos en tiempo real (integrado con OMS)     │
│  - Guías de tallas y compatibilidad                         │
│  - Preguntas frecuentes por categoría de producto           │
│  - Información de tiendas (horarios, ubicación, stock)      │
│                                                             │
│  Capacidades:                                               │
│  • "¿Hay talla M del jean azul en la tienda de Miraflores?" │
│  • "¿Cuándo llega mi pedido #78342?"                        │
│  • "¿Puedo devolver este producto si lo compré hace 40 días?"│
│  • Recomendaciones personalizadas basadas en historial      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│               ASISTENTE PARA EMPLEADOS                      │
│                                                             │
│  Canal: App interna, tablet en tienda, headset              │
│                                                             │
│  Knowledge Base:                                            │
│  - Manual de procedimientos de tienda                       │
│  - Políticas de RRHH y beneficios                           │
│  - Procedimientos de manejo de caja                         │
│  - Guía de productos: características, materiales, cuidado  │
│  - Protocolos de seguridad y emergencia                     │
│  - Scripts de ventas y objeciones frecuentes                │
│  - Reportes de inventario de su tienda                      │
│                                                             │
│  Capacidades:                                               │
│  • "¿Cómo proceso una devolución sin ticket?"               │
│  • "¿Qué materiales tiene este abrigo?"                     │
│  • "¿Cuántas unidades quedan del ref. 5521-AZ?"             │
│  • Consultas en lenguaje natural sobre procedimientos       │
└─────────────────────────────────────────────────────────────┘
```

### Arquitectura técnica del RAG

```
CONSULTA DEL USUARIO
        │
        ▼
Embedding (sentence-transformers multilingual)
        │
        ▼
Vector Store (pgvector) — índice semántico de KB
        │
        ▼
Top-K recuperados (k=5 fragmentos más relevantes)
        │
        ▼
LLM con contexto:
  - Prompt del sistema (rol: asistente de GLOBAL RETAIL NEXT)
  - Fragmentos recuperados del KB
  - Historial de conversación (últimas 5 interacciones)
  - Pregunta del usuario
        │
        ▼
Respuesta generada + fuente citada
        │
        ▼
Feedback del usuario → datos de mejora continua
```

### Personalización para el cliente

El asistente de clientes integra el perfil del usuario para enriquecer las respuestas:

```
Contexto adicional en el prompt:
  "El cliente {nombre} compró zapatillas running en marzo.
   Talla habitual: 42. Región: Lima, Perú.
   Puntos de fidelidad: 1,200. Tier: Gold."

Respuesta personalizada:
  "Hola Carlos, veo que usualmente calzas talla 42.
   La nueva colección de running que te puede interesar
   está disponible en tu talla en la tienda de Miraflores.
   Además, tienes 1,200 puntos que equivalen a S/.24 de descuento."
```

---

## P5. Pipeline MLOps Empresarial

```
FUENTES DE DATOS RETAIL
(transacciones POS, comportamiento web/app, sensores tienda,
 inventario, precios competencia, clima, eventos locales)
        │
        ▼
┌──────────────────────────────────────┐
│  INGESTA MULTICANAL                  │
│  Kafka: transacciones en tiempo real │
│  Fivetran: integración ERP/WMS/POS   │
│  Scrapers: precios competencia       │
│  APIs: clima, eventos, calendarios   │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│  DATA WAREHOUSE + FEATURE STORE      │
│  Snowflake / BigQuery                │
│  dbt (transformaciones SQL)          │
│  Feast (features para ML)            │
│                                      │
│  Features clave:                     │
│  - Historial de compras por cliente  │
│  - Elasticidad de precio por SKU     │
│  - Estacionalidad por categoría      │
│  - Score de propensión a compra      │
│  - Índice de agotamiento de stock    │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│  ENTRENAMIENTO (Kubeflow Pipelines)  │
│                                      │
│  M1: Recomendación personalizada     │
│      (Two-Tower Neural Network)      │
│  M2: Predicción de demanda           │
│      (LightGBM + Prophet)            │
│  M3: Precios dinámicos               │
│      (RL: Proximal Policy Optim.)    │
│  M4: Detección de fraude en compras  │
│      (Isolation Forest + XGBoost)    │
│  M5: Predicción de abandono (churn)  │
│      (Gradient Boosting + SHAP)      │
│  M6: Visión: análisis de estantes    │
│      (YOLO v8 fine-tuned)            │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│  REGISTRO Y VALIDACIÓN               │
│  MLflow Model Registry               │
│  Champion vs Challenger              │
│  Shadow mode: nuevo modelo en        │
│  paralelo antes de promover          │
│  Tests de equidad: ¿el modelo        │
│  recomienda igual a todos los        │
│  segmentos demográficos?             │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│  DESPLIEGUE MULTI-REGIÓN             │
│  ArgoCD + Kubernetes                 │
│  Modelos desplegados por región      │
│  (América, Europa, Asia)             │
│  Canary: 5% → 25% → 100%            │
│  Feature flags para A/B testing      │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│  MONITOREO EN PRODUCCIÓN             │
│  Evidently AI: data drift            │
│  Grafana: métricas de negocio        │
│  (CTR, conversión, ticket promedio)  │
│  Alertas: si conversión cae > 5%     │
│  → investigar drift del modelo       │
│  Reentrenamiento: semanal (demanda)  │
│  mensual (recomendación)             │
└──────────────────────────────────────┘
```

---

## P6. Blockchain para Trazabilidad Global

### El problema que resuelve en retail

Un cliente que compra una camiseta quiere saber: ¿dónde se fabricó? ¿el algodón es orgánico? ¿las condiciones laborales fueron justas? Hoy esa información no es verificable. Blockchain puede hacerla verificable e inmutable.

### Casos de uso con viabilidad real

| Caso | Descripción | Viabilidad |
|---|---|---|
| **Trazabilidad farm-to-shelf** | Registrar cada punto de la cadena: productor → fábrica → transporte → almacén → tienda | Alta — Walmart + IBM Food Trust ya lo hace en alimentos |
| **Certificados de autenticidad** | Para productos de lujo o ediciones limitadas: QR que prueba autenticidad criptográficamente | Alta — LVMH/Aura Blockchain Consortium con Dior, Louis Vuitton |
| **Certificación sostenibilidad** | Registro inmutable de que el algodón es orgánico, sin trabajo infantil | Media — requiere que proveedores adopten el sistema |
| **Smart contracts con proveedores** | Pago automático al proveedor cuando el stock llega al almacén y se verifica | Media — requiere alineamiento legal entre países |
| **Gestión de devoluciones** | Historial inmutable del ciclo de vida del producto → reduce fraude en devoluciones | Media |

### Desafíos específicos en retail global

| Desafío | Descripción |
|---|---|
| **4,000 proveedores distintos** | Cada proveedor necesita integrarse al sistema blockchain → costo y resistencia al cambio |
| **Latencia** | Un cliente en caja esperando verificación no puede esperar 30 segundos de blockchain |
| **GDPR y datos de clientes** | Los datos de compra de clientes NO van a blockchain — solo trazabilidad de productos |
| **Escalabilidad** | Millones de productos × miles de transacciones diarias → requiere Layer 2 o blockchain permisionado |

### Arquitectura recomendada

```
Hyperledger Fabric (permisionado)
Participantes: GLOBAL RETAIL NEXT + proveedores certificados
        │
        ├── Canal 1: Trazabilidad de producto (farm → tienda)
        │   Cada hito registra: quién, qué, cuándo, dónde, certificación
        │
        ├── Canal 2: Certificados de autenticidad (productos premium)
        │   QR en etiqueta → blockchain → confirma autenticidad
        │
        └── Canal 3: Smart contracts de pago a proveedores
            Trigger: RFID confirma llegada al almacén → pago automático

Cliente escanea QR del producto
        │
        ▼
App GLOBAL RETAIL NEXT consulta blockchain
        │
        ▼
"Esta camiseta de algodón orgánico fue fabricada en
 Medellín, Colombia por Textiles Verdes S.A. (certificado
 Fair Trade #45821). Llegó a Lima el 15-03-2026."
```

### Conclusión

**Conditional Go:** Blockchain para trazabilidad de productos premium y certificación de sostenibilidad tiene un caso de negocio claro (diferenciación competitiva, cumplimiento regulatorio ESG). Requiere piloto con un grupo reducido de proveedores antes de escalar a toda la cadena.

---

## P7. XR en Ventas y Marketing

### Oportunidades con evidencia de impacto

| Aplicación | Tecnología | Datos de impacto |
|---|---|---|
| **Probador virtual de ropa** | AR (cámara del móvil) | ASOS: -23% en devoluciones. Zara: +18% conversión |
| **Visualización de muebles en casa** | AR (IKEA Place app) | IKEA reporta 14x más tiempo en app vs web convencional |
| **Prueba virtual de maquillaje** | AR (Sephora Virtual Artist) | 8.5M de maquillajes probados virtualmente, +11% en ventas |
| **Showroom VR de autos / inmuebles** | VR | BMW VR showroom: +60% satisfacción de cliente pre-compra |
| **Capacitación inmersiva de vendedores** | VR | Walmart capacitó 1M+ empleados en VR; retención +75% vs video |
| **Configurador 3D de productos** | AR/Web 3D | Nike By You: clientes configuran zapatillas → mayor ticket promedio |
| **Tour virtual de tienda** | VR | Para clientes que no pueden visitar físicamente (diferente ciudad/país) |

### Oportunidades específicas para GLOBAL RETAIL NEXT

**Corto plazo (1-2 años) — AR en app móvil:**
- Probador virtual de ropa: cliente se toma foto o usa cámara en vivo, selecciona prendas y las "prueba" virtualmente
- Visualización de productos de hogar en su espacio real
- Filtros de AR en redes sociales para marketing

**Mediano plazo (2-4 años) — Kioscos XR en tienda:**
- Espejos inteligentes con AR: se ven diferentes colores/tallas sin probarse físicamente
- Asistente holográfico en kiosco para atención sin vendedor humano
- Navegación AR dentro de tienda grande: "lleva al cliente" al producto buscado

**Largo plazo (4-7 años) — Metaverso comercial:**
- Tienda virtual en metaverso con avatar del cliente
- Experiencias de marca inmersivas (lanzamientos, eventos exclusivos)

### Limitaciones a considerar

| Limitación | Impacto | Mitigación |
|---|---|---|
| No todos los clientes tienen dispositivos compatibles | Excluye segmento de bajo poder adquisitivo | Ofrecer kioscos en tienda como alternativa |
| Precisión del probador virtual varía por tipo de prenda | Prendas muy estructuradas difíciles de simular | Priorizar accesorios, maquillaje, muebles donde AR es más preciso |
| Regulación de recolección de datos biométricos | En Illinois (BIPA) y varios estados/países está regulado | No almacenar datos de imagen corporal; procesar en edge y descartar |
| Costo de contenido 3D por producto | 4,000 tiendas × miles de SKUs | Empezar con categorías de mayor margen y rotación |

---

## P8. Riesgos Regulatorios Internacionales

### Mapa regulatorio por región

| Región | Normativas clave | Riesgo principal |
|---|---|---|
| **Europa (UE)** | GDPR, AI Act 2024, Directiva Ómnibus, DSA | Análisis de comportamiento en tienda con cámaras = dato biométrico. Requiere consentimiento explícito o base legal sólida |
| **EE.UU.** | CCPA (California), BIPA (Illinois), CPRA, FTC Act | BIPA: reconocimiento facial o análisis corporal requiere consentimiento escrito. Multas de $1,000-$5,000 por violación por persona |
| **América Latina** | Ley 29733 (Perú), Ley 1581 (Colombia), LGPD (Brasil), Ley 19.628 (Chile) | Reglas distintas en cada país sobre datos personales y transferencias internacionales |
| **Asia** | PIPL (China), PDPA (Tailandia/Singapur), APPI (Japón) | China: datos de ciudadanos no pueden salir del país. Localización obligatoria. |

### Riesgos específicos de las tecnologías propuestas

| Tecnología | Riesgo regulatorio | Acción |
|---|---|---|
| **Cámaras con análisis de comportamiento** | En UE: dato biométrico si puede identificar individuos → GDPR Art. 9 | Solo análisis de tráfico agregado, sin identificación. Anónimo por diseño. |
| **IA Generativa para recomendaciones** | AI Act UE: sistemas de recomendación en plataformas grandes → riesgo "Alto". Transparencia obligatoria | Informar al usuario que recibe recomendaciones de IA. Sistema de opt-out |
| **Precios dinámicos** | Algunos países tienen leyes contra discriminación de precios por perfil. GDPR: decisiones automatizadas que afectan al individuo | No usar variables protegidas (etnia, género) en modelos de precio. Auditoría de equidad |
| **Blockchain con datos de transacciones** | GDPR: derecho al olvido vs inmutabilidad | Solo metadata en blockchain, no datos de cliente. Dato real en BD borrable |
| **Transferencia de datos entre regiones** | Cada país/bloque tiene reglas distintas | Arquitectura multiregión: datos de cada región en cloud de esa región |

### Plan de cumplimiento multi-país

```
1. DPA (Data Protection Officer) designado en UE, EE.UU. y cada país LATAM
2. DPIA (Data Protection Impact Assessment) para cada nueva tecnología
3. Registro de actividades de tratamiento actualizado por país
4. Privacy by Design en todos los sistemas desde diseño
5. Auditoría de cumplimiento trimestral por región
6. Localización de datos: datos de ciudadanos chinos en cloud China,
   datos europeos en cloud UE, etc.
```

---

## P9. Computación Cuántica en Logística

### El problema logístico de GLOBAL RETAIL NEXT

Con 4,000 tiendas en 3 continentes, la empresa enfrenta diariamente:
- Asignación óptima de inventario entre miles de tiendas
- Optimización de rutas de entrega (versión masiva del Traveling Salesman Problem)
- Planificación de reabastecimiento considerando demanda, estacionalidad y capacidad

Estos son problemas NP-hard. Las heurísticas clásicas dan soluciones buenas pero no óptimas.

### Aplicaciones cuánticas en logística

| Aplicación | Algoritmo cuántico | Estado actual | Horizonte |
|---|---|---|---|
| **Optimización de rutas de distribución** | QAOA (Quantum Approx. Optimization) | D-Wave ya vende acceso. Funciona en instancias pequeñas (< 200 variables) | 3-5 años para instancias grandes |
| **Optimización de inventario global** | QA (Quantum Annealing) | Volkswagen y Airbus lo experimentan. Resultados prometedores | 4-6 años |
| **Predicción de demanda con QML** | Quantum SVM, QNNs | Experimental en laboratorio | 7-10 años |
| **Optimización de cadena de frío** | QUBO (Quadratic Unconstrained Binary) | Fórmulas matemáticas listas, hardware limitado | 5-7 años |

### Situación actual con D-Wave

D-Wave ya ofrece **Leap Cloud** con acceso a computación cuántica. Empresas como Volkswagen optimizaron rutas de autobuses en Lisboa con 26 vehículos (instancia pequeña). Para 4,000 tiendas → instancia demasiado grande para el hardware actual.

### Recomendación

1. **Hoy:** usar algoritmos híbridos clásico-cuántico para optimización de rutas en pilotos de distribución regional (50-100 tiendas)
2. **2027-2028:** evaluar ventaja cuántica real en Amazon Braket / IBM Quantum para casos de uso específicos
3. **No invertir en hardware cuántico propio** — cloud cuántico es el camino correcto

---

## P10. Matriz Multicriterio de Decisión

| Tecnología | Madurez (0.20) | ROI esperado (0.25) | Riesgo impl. (0.20) | Cumpl. regulatorio (0.15) | Time-to-value (0.12) | Diferenciación (0.08) | **TOTAL** | **Decisión** |
|---|---|---|---|---|---|---|---|---|
| **IoT + Edge tiendas** | 9 | 9 | 8 | 8 | 9 | 7 | **8.62** | GO |
| **IA Generativa + RAG** | 7 | 9 | 7 | 6 | 8 | 8 | **7.62** | GO |
| **MLOps + Modelos predictivos** | 8 | 8 | 8 | 8 | 8 | 6 | **7.88** | GO |
| **XR (AR probadores)** | 7 | 7 | 8 | 6 | 7 | 9 | **7.20** | GO |
| **Gemelos Digitales supply chain** | 6 | 7 | 6 | 7 | 5 | 7 | **6.38** | Conditional GO |
| **Blockchain trazabilidad** | 5 | 5 | 5 | 6 | 4 | 7 | **5.17** | Conditional GO |
| **Cómputo Cuántico** | 3 | 4 | 5 | 7 | 2 | 6 | **3.97** | NO-GO |

*Escala 1-10 | El riesgo ya está invertido (10 = bajo riesgo)*

**Pesos:** Madurez=0.20, ROI=0.25, Riesgo(inv.)=0.20, Cumpl.Reg.=0.15, Time-to-value=0.12, Diferenciación=0.08

---

## P11. Business Case Integral

### TCO — 10 años

| Componente | USD (10 años) |
|---|---|
| Edge Computing: 4,000 servidores de tienda + mantenimiento | $60,000,000 |
| IoT: sensores, cámaras, RFID, beacons por tienda | $40,000,000 |
| Plataforma cloud multiregión (América, Europa, Asia) | $50,000,000 |
| IA Generativa + RAG + MLOps | $25,000,000 |
| XR: contenido 3D + dispositivos + actualización | $20,000,000 |
| Blockchain: desarrollo + integración proveedores | $12,000,000 |
| Gemelos Digitales supply chain | $15,000,000 |
| Ciberseguridad multiregión | $18,000,000 |
| Capacitación + change management | $14,000,000 |
| Soporte y operación (10 años) | $46,000,000 |
| **TOTAL TCO** | **$300,000,000** |

### ROI — Beneficios anuales proyectados

| Fuente de valor | USD/año (estado maduro, año 4+) |
|---|---|
| Reducción de devoluciones por XR (-20%) | $18,000,000 |
| Incremento de conversión por personalización IA (+15%) | $32,000,000 |
| Reducción de quiebres de stock por predicción (-30%) | $14,000,000 |
| Optimización de inventario (menos stock muerto) | $11,000,000 |
| Detección de fraude en devoluciones | $4,500,000 |
| Reducción costos logísticos (optimización rutas) | $8,000,000 |
| Eficiencia operativa tiendas (menos personal para inventario) | $7,000,000 |
| Premium de precio en productos con certificación blockchain | $5,000,000 |
| **Total beneficio anual** | **$99,500,000** |

```
ROI = ((99,500,000 × 10) - 300,000,000) / 300,000,000 × 100
    = (995,000,000 - 300,000,000) / 300,000,000 × 100
    = 231.7%

Payback period: ≈ 3.0 años (los primeros beneficios en año 1-2)
```

### VOI — Valor Intangible

| Dimensión | Beneficio |
|---|---|
| **Experiencia del cliente** | Compra omnicanal sin fricción → fidelización y NPS superior |
| **Sostenibilidad y ESG** | Blockchain certifica supply chain responsable → acceso a bonos verdes + consumidores conscientes |
| **Posicionamiento competitivo** | Referente de retail tech en 3 continentes → atracción de talento e inversión |
| **Resiliencia de supply chain** | Gemelos Digitales permiten simular disrupciones (como COVID-2020) antes de que ocurran |
| **Cumplimiento proactivo** | Sistemas preparados para nuevas regulaciones → no reacción costosa sino anticipación |

---

## P12. Hoja de Ruta — 10 Años

```
AÑO 1-2: CIMIENTOS DIGITALES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Unificar catálogo y datos de cliente en plataforma omnicanal
✓ Desplegar IoT básico en 500 tiendas piloto (sensores estantes + aforo)
✓ Edge Computing en tiendas piloto (operación offline + cámaras locales)
✓ Primer asistente RAG para empleados (app interna)
✓ Modelo de predicción de demanda en producción (reducir quiebres)
✓ Cumplimiento GDPR/LGPD/CCPA: DPA, DPIA, consent management

AÑO 3-4: PERSONALIZACIÓN E IA EN ESCALA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Asistente RAG para clientes (app, web, WhatsApp) en todas las regiones
✓ Motor de recomendación personalizada en todos los canales
✓ Probadores virtuales AR en app móvil (categorías piloto: ropa, maquillaje)
✓ Expansión Edge + IoT a las 4,000 tiendas
✓ MLOps completo: 6 modelos en producción con monitoreo automático
✓ Blockchain piloto: trazabilidad en línea premium (50 proveedores)
✓ Precios dinámicos con modelos RL en canales digitales

AÑO 5-6: EXPERIENCIA INMERSIVA + SUPPLY CHAIN INTELIGENTE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Kioscos con espejos AR en 1,000 tiendas principales
✓ Gemelos Digitales de supply chain en 3 regiones
✓ Blockchain extendido a toda la cadena de suministro certificada
✓ Capacitación VR para empleados en todos los países
✓ Showrooms VR para productos de alto valor (muebles, electrónica)
✓ Optimización de rutas híbrida cuántica-clásica en pilotos logísticos

AÑO 7-8: LIDERAZGO DE MERCADO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Gemelos Digitales de todas las tiendas y supply chain completa
✓ Asistente de compra totalmente personalizado (hiper-personalización)
✓ Plataforma de datos propia como activo monetizable (retail media)
✓ Certificación ESG basada en datos blockchain de supply chain
✓ Metaverso comercial: tiendas virtuales en plataformas emergentes

AÑO 9-10: FUTURO Y SOSTENIBILIDAD
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Reevaluación completa de tecnologías cuánticas (estado 2033-2034)
✓ Potencial ventaja cuántica en logística global si el hardware maduró
✓ Plataforma de innovación abierta con startups retail tech
✓ Modelo tecnológico exportable a socios/franquiciados
```

---

## P13. Simulación y Monitoreo de Red IoT en Python

```python
"""
Simulación de Red IoT — GLOBAL RETAIL NEXT
Simula 4,000 tiendas con múltiples sensores cada una.
Monitorea en tiempo real: temperatura, stock, aforo, energía.
Detecta anomalías automáticamente.

Conceptos demostrados:
  - Generación de datos de sensores con variación realista
  - Procesamiento de stream de eventos
  - Detección de anomalías por umbral y por estadística
  - Dashboard de monitoreo en consola
  - Alertas por tipo de sensor y severidad
"""

import random
import time
import statistics
from dataclasses import dataclass, field
from datetime import datetime
from collections import defaultdict, deque
from typing import Dict, List, Optional
import threading

# ─────────────────────────────────────────────────────────────────
# 1. MODELOS DE DATOS
# ─────────────────────────────────────────────────────────────────

@dataclass
class SensorReading:
    """Lectura de un sensor en un momento dado."""
    sensor_id: str
    tienda_id: str
    tipo: str          # temperatura, stock, aforo, energia, humedad
    valor: float
    unidad: str
    timestamp: datetime
    region: str        # america, europa, asia

@dataclass
class Alerta:
    """Alerta generada cuando un sensor supera umbrales."""
    sensor_id: str
    tienda_id: str
    tipo: str
    valor: float
    umbral: float
    severidad: str     # INFO, WARNING, CRITICAL
    mensaje: str
    timestamp: datetime

# ─────────────────────────────────────────────────────────────────
# 2. CONFIGURACIÓN DE SENSORES Y UMBRALES
# ─────────────────────────────────────────────────────────────────

CONFIGURACION_SENSORES = {
    "temperatura": {
        "unidad": "°C",
        "rango_normal": (18, 24),
        "umbral_warning": 26,
        "umbral_critical": 30,
        "descripcion": "Temperatura ambiente de tienda"
    },
    "stock": {
        "unidad": "%",
        "rango_normal": (30, 100),
        "umbral_warning": 20,
        "umbral_critical": 5,
        "descripcion": "Nivel de stock en estante (% de capacidad)"
    },
    "aforo": {
        "unidad": "personas",
        "rango_normal": (0, 80),
        "umbral_warning": 85,
        "umbral_critical": 95,
        "descripcion": "Porcentaje de ocupación de tienda"
    },
    "energia": {
        "unidad": "kWh",
        "rango_normal": (0, 150),
        "umbral_warning": 180,
        "umbral_critical": 220,
        "descripcion": "Consumo energético por hora"
    },
    "humedad": {
        "unidad": "%HR",
        "rango_normal": (40, 60),
        "umbral_warning": 70,
        "umbral_critical": 80,
        "descripcion": "Humedad relativa del ambiente"
    },
}

# ─────────────────────────────────────────────────────────────────
# 3. SIMULADOR DE TIENDA IOT
# ─────────────────────────────────────────────────────────────────

class TiendaIoT:
    """Simula los sensores de una tienda individual."""

    def __init__(self, tienda_id: str, region: str, tiene_falla: bool = False):
        self.tienda_id = tienda_id
        self.region = region
        self.tiene_falla = tiene_falla  # Simula tienda con problema
        self.sensores = self._inicializar_sensores()

    def _inicializar_sensores(self) -> Dict[str, dict]:
        """Crea sensores con valores iniciales normales."""
        sensores = {}
        for tipo, config in CONFIGURACION_SENSORES.items():
            rango = config["rango_normal"]
            valor_base = random.uniform(rango[0], rango[1])
            sensores[tipo] = {
                "sensor_id": f"{self.tienda_id}-{tipo[:3].upper()}",
                "valor_base": valor_base,
                "historial": deque(maxlen=10),  # últimas 10 lecturas
            }
        return sensores

    def leer_sensores(self) -> List[SensorReading]:
        """Genera lecturas realistas de todos los sensores."""
        lecturas = []
        ahora = datetime.now()

        for tipo, sensor in self.sensores.items():
            config = CONFIGURACION_SENSORES[tipo]
            rango = config["rango_normal"]

            # Variación normal: pequeña fluctuación alrededor del valor base
            variacion = random.gauss(0, (rango[1] - rango[0]) * 0.03)
            valor = sensor["valor_base"] + variacion

            # Si la tienda tiene falla, algunos sensores salen de rango
            if self.tiene_falla and tipo in ("temperatura", "stock"):
                if tipo == "temperatura":
                    valor = random.uniform(28, 35)  # Sobrecalentamiento
                elif tipo == "stock":
                    valor = random.uniform(0, 8)    # Casi sin stock

            # Deriva gradual: el valor base evoluciona lentamente
            sensor["valor_base"] += random.gauss(0, 0.1)
            # Mantener el valor base dentro de límites razonables
            sensor["valor_base"] = max(rango[0] * 0.5,
                                       min(rango[1] * 1.3, sensor["valor_base"]))

            valor = max(0, valor)  # No puede ser negativo
            sensor["historial"].append(valor)

            lecturas.append(SensorReading(
                sensor_id=sensor["sensor_id"],
                tienda_id=self.tienda_id,
                tipo=tipo,
                valor=round(valor, 2),
                unidad=config["unidad"],
                timestamp=ahora,
                region=self.region,
            ))

        return lecturas

# ─────────────────────────────────────────────────────────────────
# 4. MOTOR DE DETECCIÓN DE ANOMALÍAS
# ─────────────────────────────────────────────────────────────────

class DetectorAnomalias:
    """Detecta anomalías usando umbrales y análisis estadístico."""

    def __init__(self):
        self.historial_global: Dict[str, deque] = defaultdict(lambda: deque(maxlen=50))

    def analizar(self, lectura: SensorReading) -> Optional[Alerta]:
        """Evalúa una lectura y genera alerta si corresponde."""
        config = CONFIGURACION_SENSORES[lectura.tipo]
        clave = f"{lectura.tipo}"

        self.historial_global[clave].append(lectura.valor)

        # Detección por umbral absoluto
        alerta = self._detectar_por_umbral(lectura, config)
        if alerta:
            return alerta

        # Detección estadística: valor muy alejado de la media histórica
        alerta = self._detectar_por_estadistica(lectura, clave)
        return alerta

    def _detectar_por_umbral(self, lectura: SensorReading,
                              config: dict) -> Optional[Alerta]:
        valor = lectura.valor
        rango = config["rango_normal"]

        # Para stock y aforo, el problema es valor MUY BAJO o MUY ALTO
        if lectura.tipo == "stock":
            if valor <= config["umbral_critical"]:
                return self._crear_alerta(lectura, valor,
                    config["umbral_critical"], "CRITICAL",
                    f"STOCK CRÍTICO: Solo {valor:.1f}% en estante. Reposición urgente.")
            elif valor <= config["umbral_warning"]:
                return self._crear_alerta(lectura, valor,
                    config["umbral_warning"], "WARNING",
                    f"Stock bajo: {valor:.1f}%. Programar reposición.")

        elif lectura.tipo in ("temperatura", "energia", "humedad", "aforo"):
            umbral_crit = config["umbral_critical"]
            umbral_warn = config["umbral_warning"]
            if valor >= umbral_crit:
                return self._crear_alerta(lectura, valor, umbral_crit, "CRITICAL",
                    f"{lectura.tipo.upper()} CRÍTICO: {valor:.1f}{lectura.unidad}")
            elif valor >= umbral_warn:
                return self._crear_alerta(lectura, valor, umbral_warn, "WARNING",
                    f"{lectura.tipo.capitalize()} elevado: {valor:.1f}{lectura.unidad}")
        return None

    def _detectar_por_estadistica(self, lectura: SensorReading,
                                   clave: str) -> Optional[Alerta]:
        """Detecta si el valor se aleja > 3 desviaciones estándar de la media."""
        historial = list(self.historial_global[clave])
        if len(historial) < 10:
            return None
        media = statistics.mean(historial)
        stdev = statistics.stdev(historial)
        if stdev == 0:
            return None
        z_score = abs(lectura.valor - media) / stdev
        if z_score > 3.5:
            return self._crear_alerta(
                lectura, lectura.valor, media + 3.5 * stdev, "WARNING",
                f"Anomalía estadística: z-score={z_score:.1f} (valor inusual)"
            )
        return None

    def _crear_alerta(self, lectura: SensorReading, valor: float,
                      umbral: float, severidad: str, mensaje: str) -> Alerta:
        return Alerta(
            sensor_id=lectura.sensor_id,
            tienda_id=lectura.tienda_id,
            tipo=lectura.tipo,
            valor=valor,
            umbral=umbral,
            severidad=severidad,
            mensaje=mensaje,
            timestamp=lectura.timestamp,
        )

# ─────────────────────────────────────────────────────────────────
# 5. PLATAFORMA DE MONITOREO CENTRAL
# ─────────────────────────────────────────────────────────────────

class PlataformaMonitoreo:
    """
    Centro de monitoreo que recibe datos de todas las tiendas,
    detecta anomalías y muestra el dashboard en tiempo real.
    """

    def __init__(self, num_tiendas: int = 20):
        self.detector = DetectorAnomalias()
        self.alertas_activas: List[Alerta] = []
        self.estadisticas: Dict[str, dict] = defaultdict(lambda: {
            "total_lecturas": 0,
            "total_alertas": 0,
            "valores": deque(maxlen=20),
        })
        self.contadores_region = defaultdict(int)

        # Crear tiendas (algunas con falla para demostración)
        regiones = ["america", "europa", "asia"]
        self.tiendas = []
        for i in range(num_tiendas):
            region = regiones[i % 3]
            tiene_falla = (i % 7 == 0)  # ~14% de tiendas con algún problema
            tienda_id = f"T{str(i+1).zfill(4)}-{region[:3].upper()}"
            self.tiendas.append(TiendaIoT(tienda_id, region, tiene_falla))
            self.contadores_region[region] += 1

        print(f"\n{'='*65}")
        print(f"  PLATAFORMA IOT — GLOBAL RETAIL NEXT")
        print(f"  {num_tiendas} tiendas simuladas | 3 regiones | 5 sensores/tienda")
        print(f"={'='*65}")
        print(f"  Distribución: América={self.contadores_region['america']} | "
              f"Europa={self.contadores_region['europa']} | "
              f"Asia={self.contadores_region['asia']}")
        tiendas_con_falla = sum(1 for t in self.tiendas if t.tiene_falla)
        print(f"  Tiendas con falla simulada: {tiendas_con_falla}")
        print(f"{'='*65}\n")

    def procesar_ciclo(self):
        """Procesa un ciclo de lectura de todos los sensores."""
        lecturas_ciclo = []
        nuevas_alertas = []

        for tienda in self.tiendas:
            lecturas = tienda.leer_sensores()
            lecturas_ciclo.extend(lecturas)

            for lectura in lecturas:
                # Actualizar estadísticas
                key = f"{lectura.tipo}"
                self.estadisticas[key]["total_lecturas"] += 1
                self.estadisticas[key]["valores"].append(lectura.valor)

                # Detectar anomalías
                alerta = self.detector.analizar(lectura)
                if alerta:
                    nuevas_alertas.append(alerta)
                    self.estadisticas[key]["total_alertas"] += 1

        # Mantener solo las 20 alertas más recientes
        self.alertas_activas = (nuevas_alertas + self.alertas_activas)[:20]

        return lecturas_ciclo, nuevas_alertas

    def mostrar_dashboard(self, ciclo: int, lecturas: list, alertas: list):
        """Muestra el dashboard de monitoreo en consola."""
        ahora = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        print(f"\n{'─'*65}")
        print(f"  DASHBOARD IOT — Ciclo #{ciclo:04d} | {ahora}")
        print(f"{'─'*65}")

        # Resumen de sensores (promedios globales)
        print("  ESTADO DE SENSORES (promedios globales):")
        for tipo, stats in self.estadisticas.items():
            if not stats["valores"]:
                continue
            valores = list(stats["valores"])
            promedio = statistics.mean(valores)
            config = CONFIGURACION_SENSORES[tipo]
            rango = config["rango_normal"]
            # Indicador visual
            if promedio <= rango[1]:
                icono = "[OK]    "
            elif promedio <= config["umbral_warning"]:
                icono = "[WARN]  "
            else:
                icono = "[CRIT]  "
            print(f"  {icono} {tipo:<15} Promedio: {promedio:6.1f} {config['unidad']:<6} "
                  f"| Alertas: {stats['total_alertas']:3d} | "
                  f"Lecturas: {stats['total_lecturas']:5d}")

        # Alertas del ciclo actual
        if alertas:
            print(f"\n  NUEVAS ALERTAS EN ESTE CICLO ({len(alertas)}):")
            for alerta in alertas[:8]:  # Mostrar máximo 8
                icono = "!!" if alerta.severidad == "CRITICAL" else " !"
                print(f"  {icono} [{alerta.severidad:<8}] {alerta.tienda_id} "
                      f"| {alerta.tipo:<12} | {alerta.mensaje[:40]}")
        else:
            print(f"\n  Sin alertas en este ciclo.")

        # Resumen por región
        print(f"\n  LECTURAS ESTE CICLO: {len(lecturas)} | "
              f"Tiendas activas: {len(self.tiendas)}")

        # Conteo de alertas por severidad en el histórico reciente
        criticas = sum(1 for a in self.alertas_activas if a.severidad == "CRITICAL")
        warnings = sum(1 for a in self.alertas_activas if a.severidad == "WARNING")
        print(f"  ALERTAS ACTIVAS: CRITICAL={criticas} | WARNING={warnings}")
        print(f"{'─'*65}")

# ─────────────────────────────────────────────────────────────────
# 6. EJECUCIÓN PRINCIPAL
# ─────────────────────────────────────────────────────────────────

def main():
    """
    Ejecuta la simulación de la red IoT.
    Cada ciclo = lectura de todos los sensores de todas las tiendas.
    """
    NUM_TIENDAS = 20        # Reducido para la demo (en prod: 4,000)
    NUM_CICLOS = 8          # Ciclos de simulación
    INTERVALO_SEGUNDOS = 2  # Pausa entre ciclos

    plataforma = PlataformaMonitoreo(num_tiendas=NUM_TIENDAS)

    print("Iniciando simulación de red IoT...")
    print(f"Configuración: {NUM_CICLOS} ciclos con intervalo de {INTERVALO_SEGUNDOS}s\n")

    for ciclo in range(1, NUM_CICLOS + 1):
        lecturas, alertas = plataforma.procesar_ciclo()
        plataforma.mostrar_dashboard(ciclo, lecturas, alertas)

        if ciclo < NUM_CICLOS:
            time.sleep(INTERVALO_SEGUNDOS)

    # Resumen final
    print(f"\n{'='*65}")
    print("  RESUMEN FINAL DE LA SIMULACIÓN")
    print(f"{'='*65}")
    total_lecturas = sum(s["total_lecturas"] for s in plataforma.estadisticas.values())
    total_alertas = sum(s["total_alertas"] for s in plataforma.estadisticas.values())
    print(f"  Total lecturas procesadas : {total_lecturas:,}")
    print(f"  Total alertas generadas   : {total_alertas:,}")
    print(f"  Tasa de anomalías         : {(total_alertas/total_lecturas*100):.1f}%")
    print(f"  Tiendas monitoreadas      : {len(plataforma.tiendas)}")
    print(f"  Sensores por tienda       : {len(CONFIGURACION_SENSORES)}")
    print(f"  Total sensores activos    : {len(plataforma.tiendas) * len(CONFIGURACION_SENSORES)}")
    print(f"\n  En producción con 4,000 tiendas:")
    print(f"  - Total sensores : {4000 * len(CONFIGURACION_SENSORES):,}")
    print(f"  - Lecturas/min   : ~{4000 * len(CONFIGURACION_SENSORES) * 4:,} (cada 15 seg)")
    print(f"  - Kafka topics   : uno por tipo de sensor por región")
    print(f"{'='*65}")

if __name__ == "__main__":
    main()
```

### Explicación del sistema IoT — Cómo funciona

**¿Qué simula este código?**

El sistema replica el comportamiento de la infraestructura IoT real de 4,000 tiendas de GLOBAL RETAIL NEXT. Cada tienda tiene 5 tipos de sensores que generan lecturas periódicas, una plataforma central las recibe y analiza, y se generan alertas cuando algo sale de lo normal.

**Flujo de datos:**

```
Sensor físico en tienda → lectura cada 15 seg → Edge local → 
    MQTT → Kafka → Stream Processing → Dashboard central → Alerta
```

**Componentes del código y su equivalente en producción:**

| Clase/Función | Equivalente en producción |
|---|---|
| `TiendaIoT` | Servidor edge en cada tienda (NVIDIA Jetson) con sensores RFID, cámaras y termómetros |
| `SensorReading` | Mensaje MQTT publicado al broker IoT (Azure IoT Hub / AWS IoT Core) |
| `DetectorAnomalias` | Apache Flink procesando el stream de Kafka con reglas de negocio y modelos de detección |
| `PlataformaMonitoreo` | Dashboard de Grafana + reglas de alerta en PagerDuty |
| `tiene_falla=True` | Simula sensores defectuosos o tiendas con problemas reales |
| `deque(maxlen=50)` | Ventana deslizante en Flink para análisis estadístico en tiempo real |

**Detección de anomalías — dos métodos:**

1. **Por umbral absoluto:** si temperatura > 30°C → CRITICAL. Directo y determinista. Funciona para reglas de negocio conocidas.

2. **Por estadística (z-score):** si el valor se aleja más de 3.5 desviaciones estándar de la media histórica → WARNING. Detecta anomalías que las reglas no anticiparon (un sensor que deriva gradualmente sin superar el umbral absoluto).

**En producción con 4,000 tiendas:**
- 4,000 × 5 sensores = 20,000 sensores activos
- Lecturas cada 15 segundos = ~80,000 mensajes/minuto en Kafka
- Apache Flink procesa el stream en tiempo real con particionamiento por región y tipo de sensor
- Grafana muestra el mapa de calor de tiendas con alertas en tiempo real

---

## Resumen Ejecutivo — GLOBAL RETAIL NEXT

El caso de negocio es sólido: TCO de $300M en 10 años con ROI de 231% y payback en 3 años, impulsado principalmente por el incremento de conversión con personalización IA (+$32M/año) y la reducción de devoluciones con XR (-$18M/año).

**Orden de implementación:**
1. **IoT + Edge** (base de datos en tiempo real para todo lo demás)
2. **MLOps + predicción de demanda** (ROI inmediato en inventario)
3. **IA Generativa + RAG** (asistente empleados primero, clientes después)
4. **XR — probadores AR** (diferenciación competitiva, reducción de devoluciones)
5. **Gemelos Digitales supply chain** (piloto antes de escalar)
6. **Blockchain trazabilidad** (productos premium + sostenibilidad)
7. **Cómputo Cuántico** (reevaluar en 2030 según madurez del hardware)
