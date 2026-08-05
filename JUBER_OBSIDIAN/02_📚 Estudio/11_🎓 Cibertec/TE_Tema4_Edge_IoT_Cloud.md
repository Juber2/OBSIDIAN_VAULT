---
tags:
  - Cibertec
  - TecnologiasEmergentes
  - Tema4
Actualizado: 2026-06-23
Curso: "[[Tecnologias_Emergentes]]"
---

# Tema 4 — Edge/IoT y Arquitecturas Nativas Cloud

> La competitividad depende de arquitecturas capaces de integrar dispositivos, servicios y datos de forma segura, rápida y eficiente.

---

## 1. Patrones de Edge/IoT: Sensado, Ingestión y Stream Processing

### IoT vs Edge Computing

| Concepto | Definición |
|----------|------------|
| **IoT (Internet of Things)** | Red de dispositivos interconectados que generan datos y permiten la automatización de procesos. Incluye sensores, máquinas industriales, vehículos, wearables o equipos especializados. |
| **Edge Computing** | Acerca la capacidad de cómputo al lugar donde se producen los datos, reduciendo la latencia, el tráfico hacia la nube y la dependencia de conexiones permanentes. |

> Ambos conceptos habilitan la **conectividad masiva de dispositivos físicos** capaces de capturar, transmitir y procesar datos en tiempo real.

---

### Sensado: Captura y Generación de Datos

El sensado es la **etapa inicial del flujo IoT**. Los dispositivos recopilan información del entorno físico mediante sensores electrónicos, convirtiendo variables del mundo real en señales digitales que pueden interpretarse, procesarse y transmitirse.

#### Principales tipos de sensores IoT

| Tipo de sensor | Variables que mide | Aplicaciones comunes |
|----------------|--------------------|----------------------|
| **Temperatura y humedad** | Temperatura ambiente, humedad relativa | Cadena de frío, monitoreo climático, HVAC industrial |
| **Movimiento y proximidad** | Presencia, distancia, vibración | Seguridad, domótica, detección de intrusión |
| **Presión** | Presión atmosférica, de fluidos o mecánica | Industria manufacturera, sistemas hidráulicos, meteorología |
| **Luz (fotosensores)** | Intensidad lumínica, espectro | Automatización de iluminación, paneles solares, agricultura |
| **Gas y calidad del aire** | CO₂, CO, VOC, partículas | Seguridad industrial, ciudades inteligentes, edificios |
| **Acelerómetros y giroscopios** | Movimiento, orientación, velocidad angular | Wearables, vehículos autónomos, drones |
| **Cámaras e imagen** | Video, imágenes, visión computacional | Vigilancia, control de calidad industrial, reconocimiento |
| **GPS/Ubicación** | Coordenadas, trayectoria, velocidad | Logística, flotas, rastreo de activos |

> El sensado se caracteriza por su **baja latencia, alta frecuencia** y capacidad de contextualizar datos con información como tiempo, ubicación o estado del dispositivo. La calidad de esta captura es clave para asegurar precisión, confiabilidad y utilidad en cualquier arquitectura IoT.

---

### Ingestión: Transmisión y Consolidación de Datos

La ingestión es el proceso mediante el cual los datos recolectados se **transfieren desde los dispositivos o nodos Edge hacia sistemas centrales o la nube**. Requiere protocolos, herramientas y arquitecturas que garanticen conectividad confiable, segura y eficiente.

#### 4 Componentes de la ingestión

| Componente | Descripción |
|------------|-------------|
| **Protocolos de comunicación** | MQTT, CoAP, HTTP, AMQP o WebSockets. Diseñados para entornos con recursos limitados y conexiones intermitentes. MQTT es el más usado en IoT por su ligereza y bajo consumo de ancho de banda. |
| **Gateways IoT** | Dispositivos intermediarios que agregan datos, administran autenticación, realizan filtrado preliminar y optimizan el tráfico hacia la nube. Actúan como puente entre el mundo físico y los sistemas digitales. |
| **Colas y brokers de mensajería** | Kafka, RabbitMQ, Azure IoT Hub, AWS IoT Core. Permiten comunicación **asíncrona y desacoplada** entre productores y consumidores, absorbiendo picos de tráfico sin pérdida de datos. |
| **Normalización y procesamiento** | Conversión a formatos estándar como JSON, Avro o Parquet, facilitando la integración con sistemas analíticos o pipelines de datos. |

> Una ingestión efectiva actúa como **puente entre el mundo físico y el digital**, habilitando análisis en tiempo real, automatización de procesos y toma de decisiones inteligente.

---

### Stream Processing: Procesamiento en Tiempo Real

El stream processing (procesamiento de flujo) se refiere al **análisis de datos en el momento en que se generan**, sin necesidad de almacenarlos previamente. Es fundamental para escenarios donde las decisiones deben tomarse con rapidez: monitoreo industrial, sistemas de alerta o análisis de comportamiento en línea.

#### 4 Características principales

| Característica | Descripción |
|----------------|-------------|
| **Procesamiento continuo** | Los datos se analizan a medida que fluyen, sin necesidad de esperar acumulaciones o lotes. Permite obtener información actualizada y reaccionar inmediatamente a cambios en el entorno. |
| **Eventos discretos** | Cada mensaje o paquete de datos se interpreta como un **evento independiente**. Facilita la detección de patrones, anomalías o acciones específicas basadas en cada ocurrencia. |
| **Baja latencia** | Produce respuestas inmediatas o en milisegundos. Esencial para casos críticos: monitoreo industrial, detección de fraudes, sistemas autónomos o alertas en tiempo real. |
| **Escalabilidad dinámica** | El procesamiento se distribuye entre múltiples nodos o servicios que pueden aumentar o disminuir según la demanda. Garantiza estabilidad y eficiencia incluso ante grandes volúmenes. |

> El stream processing convierte flujos continuos de datos en **información accionable**: activar alertas, ejecutar reglas automáticas, alimentar dashboards en vivo o enviar información refinada hacia motores de IA o sistemas de control operativo.

---

## 2. Arquitecturas de Referencia: Serverless, Contenedores y Eventos

Las **arquitecturas cloud-native** diseñan y operan aplicaciones que aprovechan la elasticidad, automatización y servicios de la nube. Usan componentes modulares y desacoplados que facilitan la escalabilidad, el mantenimiento y la resiliencia ante fallos.

---

### Arquitecturas Serverless (Sin Servidor)

Permiten ejecutar funciones o fragmentos de código **sin necesidad de administrar servidores o infraestructura**. El proveedor cloud gestiona automáticamente el aprovisionamiento, la ejecución y la escalabilidad, cobrando solo por el tiempo de ejecución efectivo.

#### Beneficios del modelo serverless

| Beneficio | Descripción |
|-----------|-------------|
| **Sin gestión de infraestructura** | El desarrollador se enfoca solo en el código. El proveedor maneja servidores, parches, escalado y disponibilidad. |
| **Escalado automático** | La función escala de cero a miles de instancias en segundos, según la demanda real. |
| **Pago por uso** | Se cobra únicamente por el tiempo de ejecución y los recursos consumidos. No se paga por capacidad inactiva. |
| **Alta disponibilidad nativa** | Los proveedores cloud garantizan disponibilidad sin configuración adicional. |
| **Velocidad de despliegue** | Reducción significativa del tiempo desde el desarrollo hasta la puesta en producción. |
| **Reducción de costos operativos** | Elimina la necesidad de mantener servidores encendidos 24/7 para cargas esporádicas. |

> Herramientas: AWS Lambda, Azure Functions, Google Cloud Functions.

> El modelo serverless es un **componente esencial** de las arquitecturas modernas para IoT, Edge Computing y sistemas basados en eventos.

---

### Arquitecturas Basadas en Contenedores

Los contenedores son **unidades ligeras que encapsulan aplicaciones y sus dependencias** en entornos portables y reproducibles. Permiten ejecutar software de manera consistente en diferentes plataformas —nube, borde o local— sin necesidad de reconfigurar el sistema operativo.

#### Características clave de los contenedores

| Característica | Descripción |
|----------------|-------------|
| **Portabilidad** | El mismo contenedor funciona igual en desarrollo, pruebas y producción, y en cualquier cloud o entorno on-premise. |
| **Aislamiento de dependencias** | Cada contenedor lleva sus propias librerías y configuraciones, eliminando conflictos entre aplicaciones. |
| **Ligereza** | A diferencia de las máquinas virtuales, los contenedores comparten el kernel del sistema operativo, consumiendo menos recursos. |
| **Reproducibilidad** | El entorno es idéntico en cualquier plataforma, eliminando el problema "funciona en mi máquina". |
| **Orquestación con Kubernetes** | Kubernetes gestiona el despliegue, escalado, balanceo de carga y recuperación automática de contenedores a escala. |

> Los contenedores son la **base de la infraestructura moderna** para IA, IoT y microservicios, habilitando despliegues rápidos, consistentes y altamente confiables.

---

### Arquitecturas Orientadas a Eventos (EDA)

Las **Event-Driven Architectures (EDA)** se basan en que las aplicaciones **reaccionan a eventos** — acciones, cambios o señales generadas por dispositivos, usuarios o sistemas externos.

En IoT, EDA permite: procesar datos en tiempo real, detectar anomalías, activar automatizaciones, coordinar dispositivos y ejecutar modelos de IA/ML para decisiones inmediatas, todo mientras escala sin afectar el rendimiento.

#### 3 Elementos de una arquitectura orientada a eventos

| Elemento | Descripción | Ejemplos |
|----------|-------------|---------|
| **Productor de eventos** | Genera los mensajes o señales que describen una acción o cambio en el entorno. | Sensor IoT, aplicación móvil, sistema industrial, servicio backend. |
| **Canal / Broker de mensajería** | Intermediario que recibe, almacena temporalmente y distribuye los eventos a los componentes correspondientes. | Apache Kafka, RabbitMQ, MQTT, AWS EventBridge, Azure Event Hub. |
| **Consumidor de eventos** | Componente que procesa el evento y ejecuta acciones: análisis, almacenamiento, inferencia con IA o disparo de workflows. | Función serverless, contenedor, servicio backend, pipeline de datos. |

---

### Comparativa de las 3 Arquitecturas

| Criterio | Contenedores | Serverless | Orientada a eventos |
|----------|-------------|------------|---------------------|
| **Gestión de infraestructura** | Requiere configuración y operación de un orquestador (ej. Kubernetes) | Totalmente administrada por el proveedor cloud | Depende del broker de eventos y los servicios conectados |
| **Escalabilidad** | Escalado horizontal gestionado por el orquestador | Escalado automático según demanda | Escalabilidad basada en el volumen y frecuencia de eventos |
| **Costo operativo** | Moderado; se paga por uso de nodos o instancias | Bajo; se paga únicamente por las ejecuciones | Variable; depende del tráfico de eventos y la retención |
| **Latencia** | Baja y estable; ideal para cargas persistentes | Muy baja, pero puede verse afectada por *cold starts* | Variable; influida por complejidad del flujo y del broker |
| **Uso típico** | Microservicios, IA/ML, APIs, procesamiento intensivo | Funciones ligeras, automatización, validación rápida de eventos | IoT, análisis en tiempo real, detección de anomalías, automatización reactiva |

> En la práctica, es común **combinar varios enfoques** (contenedores + serverless + eventos) para construir soluciones híbridas capaces de responder a las demandas dinámicas.

---

## 3. Observabilidad, Resiliencia y Optimización de Costos en la Nube

Las arquitecturas modernas distribuidas —desde dispositivos en el borde hasta la nube— necesitan mecanismos que aseguren su estabilidad, seguridad y eficiencia. Los **tres pilares fundamentales** son: observabilidad, resiliencia y optimización de costos.

---

### Observabilidad

Capacidad de un sistema para **revelar su estado interno a partir de sus salidas externas**. A diferencia del monitoreo tradicional (centrado solo en métricas), la observabilidad permite identificar causas raíz, entender comportamientos complejos y **anticipar fallos en entornos distribuidos**.

#### 4 Componentes clave de la observabilidad

| Componente | Descripción |
|------------|-------------|
| **Métricas** | Indicadores cuantitativos sobre el comportamiento del sistema: uso de CPU, latencia, memoria, throughput, errores por segundo. |
| **Logs** | Registros detallados de eventos, transacciones y fallos que permiten reconstruir el contexto de una operación. |
| **Trazas distribuidas (tracing)** | Seguimiento del recorrido de una solicitud a través de servicios o microservicios. Clave para detectar cuellos de botella en sistemas distribuidos. |
| **Alertas y visualización** | Dashboards y reglas automatizadas que integran métricas, trazas y logs para detectar anomalías, activar alertas y facilitar el análisis operativo. |

---

### Resiliencia

Capacidad de **mantener la operación o recuperarse rápidamente** ante fallos, sin interrumpir el servicio o comprometer la disponibilidad de los datos. En entornos Edge/IoT, donde los componentes pueden operar en ubicaciones remotas o desconectadas, la resiliencia es un **requisito crítico**.

#### 5 Estrategias de resiliencia

| Estrategia | Descripción |
|------------|-------------|
| **Redundancia** | Duplicar servicios, nodos o rutas críticas para evitar puntos únicos de falla. Si un componente falla, otro lo reemplaza de forma transparente. |
| **Balanceo de carga** | Distribución inteligente del tráfico entre múltiples instancias o regiones para evitar saturación de un único nodo. |
| **Autoscaling** | Ajuste automático de recursos para responder a incrementos o disminuciones de demanda, manteniendo el rendimiento sin sobre-provisionar. |
| **Backups y replicación** | Mecanismos de protección de datos que aseguran recuperación rápida ante incidentes, desastres o pérdida de información. |
| **Pruebas de caos (chaos engineering)** | Simulación controlada de fallos en producción para validar la robustez del sistema y mejorar su capacidad de recuperación. Popularizado por Netflix (Chaos Monkey). |

> Una arquitectura resiliente **no elimina los errores**, pero mitiga su impacto, mantiene la disponibilidad del servicio y protege la integridad de la información.

---

### Optimización de Costos

Busca asegurar un **uso eficiente de los recursos cloud** sin comprometer rendimiento, disponibilidad ni seguridad. En entornos con contenedores, funciones serverless o procesamiento de eventos, los costos pueden escalar rápidamente si no se aplican mecanismos de control adecuados.

#### Estrategias principales de optimización de costos

| Estrategia | Descripción |
|------------|-------------|
| **Right-sizing** | Ajustar el tamaño de los recursos (instancias, memoria, CPU) a las necesidades reales, eliminando el sobre-aprovisionamiento. |
| **Auto-scaling** | Escalar recursos dinámicamente según la demanda, evitando mantener capacidad inactiva durante períodos de baja carga. |
| **Instancias reservadas o spot** | Usar instancias reservadas (descuento por compromiso de tiempo) o spot/preemptible (capacidad no utilizada a bajo costo) para cargas predecibles o tolerantes a interrupciones. |
| **Tiering de almacenamiento** | Clasificar datos según frecuencia de acceso (hot / warm / cold storage) y almacenar en el nivel de costo correspondiente. |
| **Monitoreo de costos (FinOps)** | Implementar dashboards de gasto, alertas de presupuesto y atribución de costos por equipo/proyecto para identificar ineficiencias. |
| **Optimización de transferencia de datos** | Minimizar el tráfico entre regiones o zonas, procesar datos en el borde (Edge) antes de enviarlos a la nube, y usar CDNs para reducir latencia y costos de egress. |

> La optimización de costos permite sostener la **eficiencia económica** del sistema sin comprometer su rendimiento ni la experiencia del usuario.

---

## 4. Gobierno y Protección del Dato en el Borde y la Nube

En los ecosistemas modernos Edge/IoT y cloud, los datos se generan, procesan y almacenan de forma **distribuida y continua**. Esto plantea desafíos en cuanto a la gestión, seguridad y control, ya que la información puede residir en múltiples ubicaciones y atravesar distintas redes y jurisdicciones.

### Data Governance (Gobierno del Dato)

Marco de **políticas, procesos y responsabilidades** para asegurar que la información se gestione de manera coherente, segura y alineada con los objetivos del negocio.

#### 5 Principios fundamentales del gobierno del dato

| Principio | Descripción |
|-----------|-------------|
| **Propiedad y responsabilidad** | Definir claramente quién es el dueño de cada conjunto de datos y quién tiene la autoridad para gestionarlo, modificarlo y establecer políticas de acceso y uso. |
| **Calidad y consistencia** | Garantizar que los datos sean precisos, completos, actualizados y consistentes entre diferentes sistemas, dispositivos y ubicaciones distribuidas. |
| **Disponibilidad controlada** | Asegurar que la información esté accesible cuando se necesite, pero únicamente para usuarios, dispositivos o aplicaciones autorizadas, bajo un modelo de **mínimos privilegios**. |
| **Trazabilidad** | Registrar todo el ciclo de vida de los datos: origen, transformaciones, movimientos, usos y eliminación. Facilita auditorías, explicabilidad y cumplimiento normativo. |
| **Cumplimiento y transparencia** | Alinear la gestión de datos con leyes de protección, regulaciones sectoriales y políticas internas, asegurando procesos transparentes y verificables. |

---

### Gobierno en Entornos Híbridos: Edge vs Cloud

#### Gobierno en el entorno Edge (borde)
Los datos se procesan **cerca de los dispositivos o sensores**, lo que introduce requisitos específicos:

- **Procesamiento local:** los datos se filtran, agregan o analizan directamente en el dispositivo o gateway, reduciendo la dependencia de la nube.
- **Protección reforzada:** controles de seguridad embebidos — cifrado local, firmware seguro y autenticación fuerte entre dispositivos.
- **Privacidad por diseño:** se minimiza la transferencia de información sensible, enviando a la nube solo lo estrictamente necesario.

#### Gobierno en el entorno Cloud
En la nube los datos se integran en plataformas centralizadas para **análisis avanzado y almacenamiento a largo plazo**:

- **Políticas unificadas:** lineamientos globales de gobierno, control de acceso, auditoría y cumplimiento normativo.
- **Escalabilidad y automatización:** servicios cloud para clasificar datos, aplicar cifrado, gestionar claves y automatizar reglas de retención o eliminación.
- **Visión consolidada:** la nube facilita la trazabilidad completa del ciclo de vida y permite aplicar modelos de análisis e inteligencia avanzada.

> La complejidad del modelo híbrido exige que las políticas de gobierno —seguridad, calidad, acceso, trazabilidad y cumplimiento— se apliquen de manera **consistente tanto en el Edge como en la nube**, asegurando que los datos mantengan su integridad y valor sin perder control operativo ni comprometer la privacidad del usuario.

---

## Ideas clave del Tema

1. La integración de **Edge e IoT** permite procesar datos en tiempo real cerca de las fuentes, lo que mejora la velocidad de respuesta, reduce la dependencia de la nube y habilita soluciones más eficientes para entornos industriales, urbanos y operativos.
2. Las **arquitecturas nativas cloud** — serverless, contenedores y sistemas orientados a eventos — ofrecen escalabilidad, resiliencia y automatización, facilitando el desarrollo de aplicaciones flexibles y altamente distribuibles.
3. La combinación de **Edge, IoT y cloud** crea un ecosistema híbrido poderoso, capaz de soportar cargas dinámicas, habilitar analítica avanzada, optimizar costos y garantizar operaciones seguras y observables a lo largo de todo el flujo tecnológico.
