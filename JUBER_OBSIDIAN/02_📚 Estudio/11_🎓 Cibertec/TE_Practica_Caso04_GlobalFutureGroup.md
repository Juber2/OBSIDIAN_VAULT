---
tags:
  - Cibertec
  - TecnologiasEmergentes
  - Practica
  - Caso04
Actualizado: 2026-06-24
Curso: "[[Tecnologias_Emergentes]]"
---

# Práctica — Caso 04: GLOBAL FUTURE GROUP

> Corporación multisectorial (banca, salud, telecomunicaciones, energía, manufactura, logística) en 40+ países. El directorio debe seleccionar 3 programas prioritarios de transformación digital para la próxima década.

---

## P1. Ubicación de Tecnologías: Hype Cycle, Tech Radar y Curva S

| Tecnología | Hype Cycle | Tech Radar | Curva S | Observación clave |
|---|---|---|---|---|
| **IA Generativa (LLMs)** | Pico → Abismo | Trial | Crecimiento acelerado | Expectativas altísimas, implementaciones en ajuste |
| **Agentes Autónomos de IA** | Pico de expectativas | Evaluar | Incubación | Muy prometedor pero aún experimental en producción |
| **RAG Corporativo** | Rampa de consolidación | Trial | Crecimiento acelerado | Casos de éxito empresariales validados |
| **Edge Computing** | Rampa de consolidación | Adoptar | Crecimiento acelerado | Maduro, ampliamente desplegado |
| **IoT Industrial (IIoT)** | Meseta de productividad | Adoptar | Madurez | Estándar en manufactura y energía líderes |
| **Cloud Native (K8s, microservicios)** | Meseta de productividad | Adoptar | Madurez | Ya es infraestructura base |
| **MLOps** | Rampa de consolidación | Adoptar | Crecimiento acelerado | Prácticas estandarizadas |
| **Blockchain (empresarial)** | Abismo de desilusión | Evaluar | Incubación | Muchos proyectos fallidos; casos nicho exitosos |
| **Web3 / DeFi** | Abismo de desilusión | Pausar/Hold | Incubación | Volatilidad regulatoria alta |
| **XR (AR/VR empresarial)** | Rampa de consolidación | Trial | Crecimiento | ROI demostrado en capacitación y mantenimiento |
| **Gemelos Digitales** | Pico de expectativas | Trial | Inicio crecimiento | Casos exitosos en manufactura; hospitalarios complejos |
| **Computación Cuántica** | Pico de expectativas | Evaluar | Incubación | Era NISQ; aplicaciones productivas a 5-10 años |

### Visualización de posicionamiento (resumen ejecutivo)

```
CURVA S — Posicionamiento por fase:

INCUBACIÓN          │ CRECIMIENTO               │ MADUREZ
────────────────────┼───────────────────────────┼──────────────────
Agentes IA          │ IA Generativa/LLMs         │ IoT Industrial
Blockchain/Web3     │ RAG Corporativo            │ Cloud Native
Computación Cuántica│ Edge Computing             │ MLOps
Gemelos Digitales   │ XR Empresarial             │
                    │                            │

HYPE CYCLE — Dónde están hoy:

Pico de expectativas:  Agentes IA, Gemelos Digitales, Cómputo Cuántico
Abismo de desilusión:  Blockchain, Web3
Rampa de consolidación: RAG, Edge, XR, MLOps
Meseta de productividad: IoT Industrial, Cloud Native
```

---

## P2. Difusión de Innovaciones y Tipping Points

### Análisis por segmento de adoptadores (Rogers)

| Tecnología | Adoptadores actuales | Distancia al Tipping Point |
|---|---|---|
| **RAG Corporativo** | Early Majority (34%) — bancos, aseguradoras, salud | **MUY CERCA** — ecosistema maduro, TCO bajo, ROI demostrado |
| **Edge + IIoT** | Early Majority → Late Majority | **CRUZANDO** — ya estándar en manufactura líder |
| **IA Generativa** | Early Adopters → Early Majority | **PRÓXIMO** — enterprise adoption acelerando en 2025-2026 |
| **XR empresarial** | Early Adopters | **En camino** — costo de hardware cayendo, ROI en capacitación probado |
| **MLOps** | Early Majority | **YA CRUZÓ** — práctica estándar en empresas data-driven |
| **Agentes IA autónomos** | Innovators (2.5%) | **LEJOS** — pocos casos en producción real, riesgo alto |
| **Blockchain empresarial** | Early Adopters estancados | **ESTANCADO** — muchos proyectos cancelados, complejidad no justificada |
| **Gemelos Digitales** | Innovators → Early Adopters | **VARIOS AÑOS** — requiere madurez IoT previa |
| **Web3** | Innovators (finanzas alternativas) | **INCIERTO** — regulación frenó adopción enterprise |
| **Computación Cuántica** | Investigadores / I+D | **LEJANO** — hardware no listo para producción |

### ¿Qué define que una tecnología está cerca del Tipping Point en GLOBAL FUTURE GROUP?

1. **Ecosistema de proveedores maduro** con SLAs enterprise disponibles
2. **Casos de uso validados en sectores similares** (banca, salud, manufactura)
3. **TCO predecible** con ROI demostrado en 2-3 años
4. **Talento disponible** en el mercado para implementar y operar
5. **Marco regulatorio** suficientemente claro para operar con seguridad

**Conclusión:** RAG Corporativo, Edge+IIoT y MLOps son las tecnologías más cerca del Tipping Point. IA Generativa está en proceso. Las demás requieren más madurez.

---

## P3. Arquitectura Empresarial Integrada

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                    CAPA DE NEGOCIO — 6 SECTORES                             ║
║  Banca │ Salud │ Telecom │ Energía │ Manufactura │ Logística                ║
╚════════════════════════════════════╤═════════════════════════════════════════╝
                                     │
╔════════════════════════════════════▼═════════════════════════════════════════╗
║                 API GATEWAY CORPORATIVO + IDENTITY FABRIC                   ║
║  Kong Enterprise │ Azure APIM │ OAuth2/OIDC │ mTLS │ Zero Trust             ║
╚══════════╤═══════════════════╤══════════════════╤══════════════════╤════════╝
           │                   │                  │                  │
╔══════════▼════════╗ ╔════════▼════════╗ ╔═══════▼══════════╗ ╔════▼═════════╗
║  PLATAFORMA       ║ ║  BLOCKCHAIN     ║ ║  GEMELOS         ║ ║  XR          ║
║  IA / RAG         ║ ║  EMPRESARIAL    ║ ║  DIGITALES       ║ ║  CORPORATIVO ║
║                   ║ ║                 ║ ║                  ║ ║              ║
║  LLM Gateway      ║ ║  Hyperledger    ║ ║  Azure Digital   ║ ║  VR training ║
║  (multi-modelo:   ║ ║  Fabric         ║ ║  Twins / NVIDIA  ║ ║  AR manten.  ║
║  Claude, GPT-4,   ║ ║  Smart contracts║ ║  Omniverse       ║ ║  MR diseño   ║
║  modelos propios) ║ ║  Trazabilidad   ║ ║  Fábricas, redes ║ ║  colaborativ.║
║                   ║ ║  Auditoría      ║ ║  hospitales      ║ ║              ║
║  RAG Corporativo  ║ ║  interempresarial║ ║  cadenas sumin.  ║ ║              ║
║  Vector Store     ║ ║                 ║ ║                  ║ ║              ║
║  Agentes IA       ║ ║                 ║ ║                  ║ ║              ║
╚══════════╤════════╝ ╚═════════════════╝ ╚══════════════════╝ ╚══════════════╝
           │
╔══════════▼══════════════════════════════════════════════════════════════════╗
║              PLATAFORMA DE DATOS Y ML — CLOUD NATIVE GLOBAL                ║
║                                                                              ║
║  Data Mesh (por dominio de negocio)    │  MLOps: Kubeflow + MLflow          ║
║  Apache Kafka (streaming global)       │  Feature Store: Feast              ║
║  Delta Lake (datos históricos)         │  Gobierno: Apache Atlas + Collibra ║
║  Kubernetes multicluster (40 países)   │  Observabilidad: Grafana + OTel    ║
║  GraphQL Federation (API de datos)     │  FinOps: Spot + Reserved           ║
║  SIEM Global: Splunk Enterprise        │  PQC: Criptografía poscuántica     ║
╚══════════╤══════════════════════════════════════════════════════════════════╝
           │  Seguridad: Zero Trust + IEC 62443 (OT) + ISO 27001
╔══════════▼══════════════════════════════════════════════════════════════════╗
║              CAPA EDGE + IOT INDUSTRIAL (por sector)                       ║
║                                                                              ║
║  Manufactura: PLCs, robots, sensores de calidad, visión industrial          ║
║  Energía: SCADA, subestaciones, turbinas, medidores inteligentes            ║
║  Salud: equipos médicos, wearables, sensores ambientales hospitalarios      ║
║  Logística: GPS flotas, RFID almacenes, drones de inventario               ║
║  Telecom: antenas 5G con cómputo edge, nodos de red distribuidos            ║
║  Banca: ATMs inteligentes, sensores de seguridad física de sucursales       ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### Principios de arquitectura para 40 países

| Principio | Implementación |
|---|---|
| **Data Mesh** | Cada sector (banca, salud, etc.) es dueño de sus datos. Plataforma central proporciona infraestructura. Datos accesibles como productos |
| **Zero Trust** | Ninguna red es confiable. Todo se verifica. mTLS entre servicios, RBAC granular |
| **Soberanía de datos** | Datos de cada país procesados en cloud de esa región. China, UE, EE.UU. tienen stacks separados |
| **Multi-cloud antifrágil** | AWS + Azure + GCP según sector y país. Sin dependencia de un solo proveedor |
| **API-first** | Todos los sistemas exponen APIs. Los sectores se integran por APIs, no por bases de datos compartidas |

---

## P4. Pipeline MLOps Global con Gobierno de Modelos

```
FUENTES DE DATOS (por sector y región)
Banca: transacciones, fraude, crédito
Salud: EHR, imágenes médicas, señales vitales
Energía: SCADA, medidores, clima
Manufactura: sensores de planta, calidad, mantenimiento
Logística: GPS, RFID, demanda
Telecom: tráfico de red, fallos, NPS
        │
        ▼
╔═══════════════════════════════════════════════╗
║          INGESTA FEDERADA                     ║
║  Kafka por sector + región                    ║
║  Conectores: Debezium (CDC), Fivetran         ║
║  Validación: Great Expectations               ║
║  Schema Registry: Confluent                   ║
╚═══════════════════════╤═══════════════════════╝
                        │
                        ▼
╔═══════════════════════════════════════════════╗
║       FEATURE STORE CORPORATIVO               ║
║  Feast (online + offline)                     ║
║  Features compartidos entre sectores:         ║
║  - Score de cliente (banca + retail)          ║
║  - Predicción de demanda (energía + logística)║
║  - Riesgo operativo (manufactura + salud)     ║
╚═══════════════════════╤═══════════════════════╝
                        │
                        ▼
╔═══════════════════════════════════════════════╗
║       ENTRENAMIENTO DISTRIBUIDO               ║
║  Kubeflow Pipelines (por dominio)             ║
║  GPU clusters por región                      ║
║  Federated Learning (datos no centralizables) ║
║  → Banca: modelos de fraude sin mover datos   ║
║  → Salud: modelos clínicos sin mover PHI      ║
╚═══════════════════════╤═══════════════════════╝
                        │
                        ▼
╔═══════════════════════════════════════════════╗
║       GOBIERNO DE MODELOS (MODEL GOVERNANCE)  ║
║  MLflow Model Registry corporativo            ║
║                                               ║
║  Por cada modelo:                             ║
║  ✓ Ficha de modelo (Model Card)               ║
║  ✓ Dataset de entrenamiento versionado        ║
║  ✓ Métricas de equidad (fairness audit)       ║
║  ✓ Pruebas de adversarial robustness          ║
║  ✓ Aprobación de comité (por riesgo)          ║
║    → Bajo riesgo: aprobación técnica          ║
║    → Alto riesgo (crédito, diagnóstico):      ║
║      aprobación comité ético + legal          ║
║  ✓ Fecha de revisión obligatoria              ║
║  ✓ Plan de retiro del modelo                  ║
╚═══════════════════════╤═══════════════════════╝
                        │
                        ▼
╔═══════════════════════════════════════════════╗
║       DESPLIEGUE MULTI-REGIÓN                 ║
║  ArgoCD + Kubernetes (multicluster)           ║
║  Canary per region: empieza en 1 país         ║
║  Shadow mode: nuevo modelo en paralelo        ║
║  Feature flags por sector y país             ║
╚═══════════════════════╤═══════════════════════╝
                        │
                        ▼
╔═══════════════════════════════════════════════╗
║       MONITOREO GLOBAL                        ║
║  Evidently AI: data drift por sector          ║
║  Grafana: métricas de negocio por modelo      ║
║  SHAP: explicabilidad en prod para auditoría  ║
║  Alertas: slack/pagerduty por umbral          ║
║  Reentrenamiento: automático o manual         ║
║  según nivel de riesgo del modelo             ║
╚═══════════════════════════════════════════════╝
```

### Federated Learning — por qué es clave para GLOBAL FUTURE GROUP

```
Problema: los datos de salud de pacientes en Chile NO pueden
          compartirse con el servidor central en EE.UU.

Solución Federated Learning:
  Servidor central envía el modelo base a cada país
          │
          ▼
  Cada país entrena localmente con sus datos (nunca salen)
          │
          ▼
  Solo los gradientes (actualizaciones del modelo) se envían al centro
          │
          ▼
  El servidor agrega los gradientes → modelo global mejorado
          │
          ▼
  Modelo global redistribuido a todos los países

Resultado: modelo que aprendió de datos de 40 países
           sin que un solo dato cruzara una frontera
```

---

## P5. Análisis Integral de Riesgos

### Riesgos Técnicos

| Riesgo | Sectores más expuestos | Mitigación |
|---|---|---|
| **Alucinaciones de LLMs en decisiones críticas** | Banca (crédito), Salud (diagnóstico) | RAG con base validada + supervisión humana obligatoria en decisiones de alto impacto |
| **Interoperabilidad entre sistemas heredados** | Todos | API Gateway + capa de abstracción FHIR (salud), ISO 20022 (banca) |
| **Vendor lock-in cloud** | Todos | Multi-cloud estratégico + Kubernetes para portabilidad |
| **Deuda técnica en centros de datos heredados** | Manufactura, Banca | Estrategia gradual: strangler fig pattern para migración sin big bang |
| **Latencia en edge con modelos grandes** | Manufactura, Energía | Modelos cuantizados (GGUF, INT8) para inferencia edge |
| **Ataques a infraestructura OT** | Energía, Manufactura | IEC 62443, segmentación OT/IT, IDS industrial |

### Riesgos Operativos

| Riesgo | Mitigación |
|---|---|
| **Resistencia cultural en 40 países** | Change management localizado. Champions internos por país. No imponer desde centro |
| **Brecha de talento digital** | Academia corporativa + convenios universidades + reskilling del personal existente |
| **Dependencia de proveedores críticos (OpenAI, AWS)** | Multi-proveedor + modelos open-source de respaldo (Mistral, LLaMA) |
| **Complejidad de coordinación global** | Centro de Excelencia Digital (CoE) con representantes de cada sector y región |
| **Cambios regulatorios que invalidan implementaciones** | Arquitectura modular: cambiar un componente sin reescribir todo |

### Riesgos Éticos

| Riesgo | Descripción | Mitigación |
|---|---|---|
| **Sesgos en modelos de crédito** | Modelo niega crédito a grupos vulnerables desproporcionalmente | Fairness audit trimestral. Métricas de equidad por grupo demográfico |
| **IA en diagnóstico médico sin supervisión** | Error del modelo causa daño a paciente | LLM/modelo = herramienta de apoyo. Médico siempre decide |
| **Vigilancia laboral excesiva con IoT** | Sensores rastrean movimiento de empleados de forma invasiva | Política de datos de empleados. Anonimización. Consulta sindical |
| **Deepfakes y desinformación con IA gen.** | Uso malicioso de capacidades generativas | Política de uso aceptable + watermarking de contenido generado |
| **Agentes autónomos sin supervisión** | Agente toma decisiones financieras o médicas sin control humano | Human-in-the-loop obligatorio para decisiones de alto impacto |

### Riesgos Legales y Regulatorios

| Ámbito | Riesgo | Acción |
|---|---|---|
| **AI Act (UE 2024)** | Sistemas de IA de alto riesgo en banca/salud sin certificación | Clasificar cada sistema de IA. Certificar los de alto riesgo antes del deadline |
| **GDPR + leyes locales (40 países)** | Procesamiento de datos personales sin base legal adecuada | DPA en cada jurisdicción. Mapeo de flujos de datos. Consentimiento granular |
| **Regulación financiera** | Algoritmos de crédito sin explicabilidad (derecho a explicación, GDPR Art. 22) | SHAP obligatorio en todos los modelos de crédito y scoring |
| **Regulación de salud** | IA médica sin aprobación como dispositivo médico | Clasificación MDR (UE) o FDA (EE.UU.) según uso clínico |
| **Ciberseguridad** | NIS2 (UE), NERC CIP (energía), SOX (financiero) | Auditorías de cumplimiento anuales por sector |
| **Soberanía digital** | China exige datos locales. Rusia restringe servicios cloud extranjeros | Stack tecnológico separado por países con regulación estricta |

---

## P6. Blockchain: Cuándo Sí y Cuándo NO

### Cuándo Blockchain SÍ aporta valor en GLOBAL FUTURE GROUP

| Caso | Sector | Justificación |
|---|---|---|
| **Trazabilidad de cadena de suministro** | Manufactura, Logística | Registro inmutable de origen, procesamiento y entrega de materiales |
| **Liquidación interbancaria** | Banca | Smart contracts reducen tiempo de liquidación de T+2 a segundos |
| **Certificados de energía renovable (RECs)** | Energía | Evita doble contabilización, inmutable y verificable |
| **Historial de mantenimiento de equipos** | Manufactura | Registro que no puede alterarse → valor para auditorías y seguros |
| **Autenticidad de medicamentos** | Salud | Verificar que un medicamento no es falsificado en cada punto de la cadena |
| **Identidad descentralizada (DID) corporativa** | Todos | Empleados y socios con credenciales verificables sin repositorio central |

### Cuándo Blockchain NO debería utilizarse

| Situación | Por qué NO usar Blockchain |
|---|---|
| **Base de datos centralizada con un único administrador** | Si solo una entidad controla los datos, una BD tradicional es más eficiente y barata |
| **Datos que deben poder modificarse o borrarse** | Blockchain es inmutable. GDPR exige derecho al olvido. Conflicto estructural |
| **Alta velocidad de transacciones (tiempo real)** | Latencia de blockchain (segundos) incompatible con trading de alta frecuencia o SCADA |
| **Datos confidenciales que nunca deben compartirse** | El registro en blockchain es visible para todos los nodos de la red |
| **Proyecto sin múltiples partes que necesiten confiar entre sí** | Blockchain resuelve el problema de confianza entre partes. Sin ese problema, es overhead innecesario |
| **Almacenamiento de grandes volúmenes de datos** | Imágenes médicas, video de cámaras, telemetría masiva → inviable en blockchain |
| **Cuando la regulación no está clara** | Web3/DeFi en sectores financieros regulados → riesgo legal sin compensación clara |

### Regla de oro para blockchain en GLOBAL FUTURE GROUP

```
Antes de proponer blockchain, responder:
  1. ¿Hay múltiples partes que necesiten confiar entre sí sin intermediario? → Si NO: usar BD tradicional
  2. ¿Los datos deben ser inmutables y auditables? → Si NO: usar BD tradicional
  3. ¿El volumen y velocidad son compatibles con blockchain? → Si NO: blockchain no aplica
  4. ¿La regulación lo permite? → Si NO: esperar claridad regulatoria

Solo si las 4 respuestas son SÍ: evaluar blockchain
```

---

## P7. XR y Gemelos Digitales Empresariales

### XR por Sector en GLOBAL FUTURE GROUP

| Sector | Aplicación XR | Tecnología | Impacto medible |
|---|---|---|---|
| **Manufactura** | Mantenimiento asistido con AR | AR (HoloLens 2) | -35% tiempo de diagnóstico, -25% errores en reparación |
| **Manufactura** | Capacitación en maquinaria peligrosa | VR | -40% accidentes en formación, +60% retención |
| **Energía** | Inspección virtual de subestaciones | AR + drones | Técnico remoto guía inspección → -70% desplazamientos |
| **Salud** | Formación quirúrgica inmersiva | VR | -67% errores en procedimientos (JAMA Surgery 2023) |
| **Logística** | Picking en almacén con AR | AR (smart glasses) | +25% eficiencia en picking (DHL, 2022) |
| **Banca** | Capacitación en atención al cliente | VR | +75% retención vs capacitación en video |
| **Telecom** | Diseño de infraestructura de red | MR | Visualizar cobertura 5G antes de instalar antenas |

### Gemelos Digitales por Sector

| Sector | Gemelo Digital | Beneficio |
|---|---|---|
| **Manufactura** | Línea de producción completa | Optimización de turnos, mantenimiento predictivo, simulación de cambios antes de aplicar |
| **Energía** | Red eléctrica de distribución | Simulación de demanda pico, planificación de expansión, respuesta a fenómenos climáticos |
| **Logística** | Centro de distribución / almacén | Optimización de layout, simulación de flujo de pedidos, reducción de cuellos de botella |
| **Salud** | Hospital completo | Simulación de ocupación UCI, planificación de expansión, respuesta a pandemias |
| **Telecom** | Red de antenas y backhaul | Optimización de cobertura, simulación de fallos, planificación de 5G/6G |
| **Banca** | Red de sucursales y ATMs | Optimización de ubicaciones, simulación de demanda por horario y zona |

### Sinergia XR + Gemelos Digitales

```
Gemelo Digital del Motor de Turbina (Manufactura)
        │
        ├── Datos en tiempo real de sensores IoT → estado actual
        ├── Historial de mantenimiento → predicción de falla
        │
        ▼
Técnico con HoloLens (AR)
        │
        ├── Ve el gemelo digital superpuesto sobre la turbina real
        ├── El gemelo muestra: temperatura interna, vibración, desgaste
        ├── IA sugiere: "Cambiar rodamiento #3 en los próximos 7 días"
        └── Técnico sigue instrucciones paso a paso en visión aumentada
```

Esta combinación reduce el tiempo de diagnóstico de horas a minutos y permite que técnicos junior realicen trabajos que antes requerían expertos senior.

---

## P8. Computación Cuántica — Oportunidades Reales y Limitaciones

### Estado actual (2026) — Era NISQ

| Hardware | Qubits | Empresa | Estado |
|---|---|---|---|
| IBM Heron | 133 qubits | IBM | Disponible en cloud (IBM Quantum) |
| Google Willow | 105 qubits | Google | Investigación + acceso limitado |
| D-Wave Advantage | 5,000+ qubits (annealing) | D-Wave | Leap Cloud — optimización combinatoria |
| IonQ Forte | 36 qubits (alta calidad) | IonQ | AWS Braket + Azure Quantum |

**Limitación crítica:** los qubits actuales son físicos y ruidosos. Para aplicaciones reales se necesitan qubits *lógicos* (con corrección de errores), lo que requiere ~1,000 qubits físicos por qubit lógico. Hoy: decenas de qubits lógicos disponibles.

### Aplicaciones por sector de GLOBAL FUTURE GROUP

| Sector | Aplicación | Estado | Horizonte |
|---|---|---|---|
| **Banca** | Optimización de portafolio (Markowitz cuántico) | Experimentos en D-Wave | 3-5 años |
| **Banca** | Simulación de riesgo Monte Carlo cuántico | Algoritmos listos, hardware limitado | 4-6 años |
| **Logística** | Optimización de rutas (VRP cuántico) | D-Wave Leap — instancias pequeñas | 3-5 años |
| **Manufactura** | Optimización de scheduling de producción | QAOA en investigación | 5-7 años |
| **Salud** | Descubrimiento de fármacos (simulación molecular) | IBM + Merck investigan | 7-10 años |
| **Energía** | Optimal Power Flow cuántico | Algoritmos en laboratorio | 5-8 años |
| **Todos** | **Criptografía poscuántica** | **DISPONIBLE HOY** | **Inmediato** |

### La amenaza "Harvest Now, Decrypt Later"

```
Situación actual:
  Actores maliciosos (estados-nación) capturan datos cifrados de GLOBAL FUTURE GROUP HOY
  Guardan esos datos cifrados durante 5-10 años
  Cuando exista computadora cuántica suficiente → descifran todo

Datos en riesgo:
  - Secretos bancarios a largo plazo
  - Registros médicos permanentes
  - Contratos de energía y manufactura confidenciales
  - Comunicaciones ejecutivas

Solución urgente: migrar a NIST PQC 2024
  - ML-KEM (Kyber): para intercambio de claves
  - ML-DSA (Dilithium): para firmas digitales
  - SLH-DSA (SPHINCS+): firma alternativa
```

### Recomendación para GLOBAL FUTURE GROUP

1. **Hoy (urgente):** migrar comunicaciones críticas a criptografía poscuántica (NIST PQC 2024)
2. **2026-2028:** experimentos con D-Wave para optimización de rutas logísticas y portafolios bancarios (bajo costo, alto aprendizaje)
3. **2028-2030:** reevaluar según estado del hardware. Si IBM/Google demuestran ventaja cuántica en casos de negocio reales → escalar
4. **No construir hardware cuántico propio** — cloud cuántico es el modelo correcto para empresas

---

## P9. Matriz Multicriterio Global

*Escala 1-10 | Riesgo y Complejidad invertidos (10 = bajo riesgo/complejidad = mejor)*

| Tecnología | ROI (0.22) | TCO relativo (0.18) | VOI (0.18) | Riesgo inv. (0.16) | Madurez (0.16) | Complejidad inv. (0.10) | **TOTAL** |
|---|---|---|---|---|---|---|---|
| **RAG Corporativo Global** | 9 | 9 | 8 | 8 | 8 | 8 | **8.52** |
| **Edge + IIoT Multi-sector** | 9 | 7 | 9 | 8 | 9 | 6 | **8.26** |
| **MLOps + IA Predictiva** | 9 | 8 | 8 | 8 | 8 | 7 | **8.26** |
| **IA Generativa empresarial** | 8 | 7 | 9 | 6 | 7 | 7 | **7.52** |
| **XR empresarial (cap. + mant.)** | 7 | 7 | 8 | 8 | 7 | 7 | **7.38** |
| **Gemelos Digitales** | 7 | 5 | 8 | 6 | 6 | 5 | **6.38** |
| **Agentes IA autónomos** | 7 | 7 | 8 | 4 | 4 | 4 | **5.84** |
| **Blockchain empresarial** | 5 | 6 | 6 | 5 | 5 | 4 | **5.18** |
| **Criptografía Poscuántica** | 6 | 8 | 9 | 9 | 9 | 8 | **7.90** |
| **Web3 / DeFi** | 4 | 6 | 5 | 3 | 3 | 3 | **3.98** |
| **Computación Cuántica (prod.)** | 3 | 7 | 6 | 5 | 3 | 3 | **4.24** |

### Clasificación por umbral

| Score | Decisión | Tecnologías |
|---|---|---|
| ≥ 8.0 | **GO — Programa prioritario** | RAG Corporativo, Edge+IIoT, MLOps |
| 7.0-7.9 | **GO — Iniciativa complementaria** | PQC (urgente), IA Generativa, XR |
| 5.5-6.9 | **Conditional GO — Piloto** | Gemelos Digitales, Agentes IA |
| < 5.5 | **NO-GO por ahora** | Blockchain (caso por caso), Web3, Cuántico producción |

---

## P10. Las Tres Iniciativas Prioritarias

El directorio exige seleccionar **3 programas** para la primera fase. Con base en la matriz multicriterio:

### PROGRAMA 1: RAG CORPORATIVO GLOBAL (Score: 8.52)

**¿Por qué primero?**

- **ROI inmediato:** en los 6 sectores, el conocimiento corporativo está fragmentado en miles de documentos, manuales, regulaciones y procedimientos. RAG hace ese conocimiento accesible a todos los empleados en segundos.
- **Habilitador de otros programas:** los datos y la plataforma RAG alimentan a los agentes IA futuros.
- **Bajo riesgo:** tecnología madura, TCO predecible, sin impacto regulatorio de alto riesgo.
- **Alta adopción esperada:** los empleados lo adoptan porque resuelve una frustración diaria real (encontrar información).

**Alcance del programa:**
- LLM Gateway corporativo (multi-modelo: Claude, GPT-4, modelos open source)
- Knowledge base por sector con actualización automática
- Asistente de campo para manufactura y energía (con modo offline)
- Asistente regulatorio para banca (normativas de 40 países indexadas)
- Asistente clínico para salud (guías médicas validadas)
- Inversión estimada: $35M en 3 años

### PROGRAMA 2: EDGE + IIOT MULTI-SECTOR (Score: 8.26)

**¿Por qué segundo?**

- **Es la infraestructura de datos del futuro:** sin datos en tiempo real de fábricas, hospitales, redes eléctricas y flotas, los gemelos digitales, los modelos predictivos y los agentes IA no tienen sobre qué operar.
- **ROI directo en manufactura y energía:** mantenimiento predictivo con IoT tiene el payback más rápido y medible de todas las iniciativas.
- **Tecnología madura:** IoT industrial es estándar en las industrias líderes. No es experimental.
- **Resiliencia operativa:** edge permite que las operaciones continúen incluso sin conectividad cloud.

**Alcance del programa:**
- Despliegue de nodos edge en todas las plantas, subestaciones y centros logísticos
- Estandarización de protocolos: MQTT, OPC-UA, FHIR según sector
- Plataforma Kafka + stream processing unificada
- Primeros modelos de mantenimiento predictivo por sector
- Inversión estimada: $55M en 3 años

### PROGRAMA 3: MLOPS + IA PREDICTIVA GLOBAL (Score: 8.26)

**¿Por qué tercero?**

- **Multiplica el valor de los otros dos programas:** los datos del IIoT + el conocimiento del RAG + modelos ML = inteligencia operativa real.
- **ROI medible en todos los sectores:** detección de fraude (banca), predicción de sepsis (salud), falla de equipos (manufactura), demanda energética (energía), rutas óptimas (logística), churn de clientes (telecom).
- **Gobierna todos los modelos:** sin MLOps, los modelos de IA se vuelven cajas negras inauditables — riesgo regulatorio y ético.
- **Federated Learning resuelve la soberanía de datos:** permite entrenar modelos globales sin mover datos entre países.

**Alcance del programa:**
- Plataforma MLOps unificada (Kubeflow + MLflow) con Feature Store corporativo
- 12 modelos iniciales en producción (2 por sector)
- Marco de gobierno de modelos con Model Cards y comité de ética
- Federated Learning para sectores con restricciones de datos (banca, salud)
- Inversión estimada: $40M en 3 años

---

## P11. Business Case Ejecutivo para el Directorio

### Resumen de Inversión — Tres Programas (3 años, fase 1)

| Programa | Inversión 3 años | Beneficio anual (año 3+) | Payback |
|---|---|---|---|
| RAG Corporativo Global | $35,000,000 | $28,000,000 | 15 meses |
| Edge + IIoT Multi-sector | $55,000,000 | $62,000,000 | 11 meses |
| MLOps + IA Predictiva | $40,000,000 | $45,000,000 | 13 meses |
| **TOTAL** | **$130,000,000** | **$135,000,000/año** | **~13 meses** |

### Desglose de beneficios por sector y programa

**RAG Corporativo — $28M/año:**
- Reducción tiempo de búsqueda de información: -60% → productividad +$12M
- Reducción errores por procedimiento incorrecto → $6M
- Capacitación más rápida de nuevos empleados → $5M
- Cumplimiento regulatorio más ágil → $5M

**Edge + IIoT — $62M/año:**
- Mantenimiento predictivo (evitar fallas costosas): $22M
- Reducción de tiempo de inactividad no planificado (-40%): $18M
- Optimización energética en plantas e instalaciones: $8M
- Detección de fraude en medidores y consumos: $6M
- Logística: reducción de mermas y pérdidas: $8M

**MLOps + IA Predictiva — $45M/año:**
- Detección de fraude bancario (+$15M recuperado)
- Predicción de churn en telecom → retención (+$8M ingresos)
- Reducción de rechazos de crédito por sesgo del modelo (VOI): $5M
- Optimización de inventario y cadena de suministro: $9M
- Diagnóstico temprano en salud → reducción de costos hospitalarios: $8M

### ROI Consolidado

```
Inversión total 3 programas: $130,000,000
Beneficio anual (estado maduro, año 3): $135,000,000

ROI 5 años = ((135,000,000 × 5) - 130,000,000) / 130,000,000 × 100
           = (675,000,000 - 130,000,000) / 130,000,000 × 100
           = 419%

Payback promedio: ~13 meses
```

### VOI — Valor Estratégico No Financiero

| Dimensión | Impacto |
|---|---|
| **Posicionamiento competitivo** | Líder tecnológico en 6 sectores en 40 países |
| **Resiliencia organizacional** | Operaciones que no dependen de un proveedor o infraestructura |
| **Cumplimiento regulatorio proactivo** | Sistemas preparados para AI Act, GDPR y regulaciones futuras |
| **Sostenibilidad ESG** | Optimización energética, reducción de desperdicio, datos para reportes ESG |
| **Atracción de talento** | Empresa tecnológicamente atractiva para los mejores perfiles digitales |
| **Capacidad de innovación** | Plataforma que habilita futuras iniciativas (Gemelos Digitales, Agentes IA) |

---

## P12. Roadmap 10 Años: KPIs, OKRs, Presupuesto, Riesgos y Gestión del Cambio

### Fases del Roadmap

```
FASE 1 (Año 1-3): FUNDACIÓN Y VALOR INMEDIATO — $130M
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Programa 1: RAG Corporativo Global (full rollout)
✓ Programa 2: Edge + IIoT (manufactura, energía, logística)
✓ Programa 3: MLOps + IA Predictiva (12 modelos en prod.)
✓ Ciberseguridad base: Zero Trust + PQC en comunicaciones críticas
✓ Centro de Excelencia Digital (CoE) establecido
✓ Data Mesh corporativo: gobernanza de datos unificada
✓ Federated Learning para sectores con restricciones de datos

FASE 2 (Año 4-6): INTELIGENCIA AVANZADA — $90M
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ IA Generativa empresarial en todos los sectores
✓ Agentes IA autónomos (alcance limitado + supervisión humana)
✓ XR: capacitación y mantenimiento en manufactura, energía, salud
✓ Gemelos Digitales: plantas industriales y redes de energía
✓ Blockchain: trazabilidad en manufactura y certificados renovables
✓ 30+ modelos ML en producción con gobierno maduro

FASE 3 (Año 7-8): DIFERENCIACIÓN ESTRATÉGICA — $60M
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Gemelos Digitales de toda la corporación
✓ Agentes IA en workflows de mayor autonomía (con supervisión)
✓ XR en todos los sectores + realidad mixta para colaboración
✓ Plataforma de datos como activo comercializable
✓ Primeros experimentos cuánticos en logística y finanzas

FASE 4 (Año 9-10): LIDERAZGO E INNOVACIÓN — $40M
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Reevaluación completa de tecnologías cuánticas
✓ Plataforma de innovación abierta con ecosistema de startups
✓ Exportar el modelo tecnológico como oferta B2B a terceros
✓ Posible ventaja cuántica en logística y finanzas si hardware maduró
```

**Presupuesto total 10 años:** $320M

### OKRs por Fase

**FASE 1 — OKRs:**
- O1: Lograr que el 80% de empleados use el asistente RAG activamente
  - KR1: 50,000 empleados onboarded en RAG en año 1
  - KR2: 85% satisfacción en encuesta de uso trimestral
  - KR3: Reducción de 60% en tiempo promedio de búsqueda de información
- O2: Digitalizar la operación industrial con IIoT
  - KR1: 90% de equipos críticos monitoreados en tiempo real al año 2
  - KR2: Reducir fallas no planificadas en 35% al año 3
  - KR3: MAPE < 8% en modelos de predicción de demanda
- O3: Establecer gobierno de IA confiable
  - KR1: 100% de modelos de alto riesgo con Model Card aprobada
  - KR2: Cero incidentes regulatorios relacionados con IA en 3 años
  - KR3: Auditoría externa de equidad superada en todos los modelos críticos

### KPIs Operativos Globales

| Categoría | KPI | Meta (año 3) |
|---|---|---|
| **Eficiencia operativa** | Reducción de tiempo de inactividad no planificado | -35% |
| **IA/RAG** | Tasa de adopción activa del asistente | >75% empleados |
| **IA/RAG** | Precisión de respuestas RAG (vs experto) | >90% |
| **IIoT** | % equipos críticos monitoreados | >90% |
| **MLOps** | Modelos en producción con monitoreo activo | 100% |
| **Seguridad** | Tiempo medio de detección de incidente (MTTD) | <4 horas |
| **Financiero** | ROI acumulado del programa | >200% a 5 años |
| **Financiero** | Reducción de costos operativos | -18% a año 3 |
| **Talento** | % empleados con certificación digital | >60% a año 2 |
| **ESG** | Reducción huella de carbono por optimización | -12% a año 3 |

### Gestión del Cambio — Plan Global

```
DIMENSIÓN 1: LIDERAZGO
  - C-Suite alineado con la visión → mensajes consistentes en 40 países
  - Champions locales por país y sector → 200+ Digital Champions formados
  - Comité de Transformación Digital con reunión mensual

DIMENSIÓN 2: COMUNICACIÓN
  - Campaña interna "Global Future Group Digital" multiidioma
  - Casos de éxito compartidos entre países cada mes
  - Dashboard público de progreso accesible a todos los empleados

DIMENSIÓN 3: CAPACITACIÓN
  - Academia Digital corporativa con rutas por rol y sector
  - Programas de reskilling para roles que cambian con IA
  - Alianzas con Coursera, Udacity, universidades locales
  - 50,000 horas de capacitación digital en año 1

DIMENSIÓN 4: CULTURA
  - Hackathons semestrales de innovación por sector
  - Programa de reconocimiento para primeros adoptadores
  - Tolerancia al error en pilotos → aprendizaje sin penalización
```

---

## P13. Sostenibilidad Tecnológica y Financiera

### Sostenibilidad Tecnológica

| Mecanismo | Descripción |
|---|---|
| **Arquitectura modular** | Cada componente puede reemplazarse sin reescribir el sistema entero. Evita grandes renovaciones costosas |
| **Estándares abiertos** | FHIR, ISO 20022, OPC-UA, OpenAPI → evita dependencia de formatos propietarios |
| **Multi-proveedor** | Nunca más del 40% de dependencia en un solo proveedor cloud o de IA |
| **Deuda técnica controlada** | KPI de deuda técnica por sistema. Presupuesto reservado para modernización continua (15% del presupuesto tecnológico anual) |
| **Centro de Excelencia Digital** | Equipo interno que acumula conocimiento → no depender 100% de consultoras externas |
| **Revisiones tecnológicas anuales** | Tech Radar propio de GLOBAL FUTURE GROUP actualizado cada 6 meses |

### Sostenibilidad Financiera

| Mecanismo | Descripción |
|---|---|
| **FinOps (Cloud Financial Management)** | Dashboards de gasto cloud por sector. Alertas de desviación. Spot instances para cargas no críticas |
| **Autofinanciamiento con ahorros** | Los ahorros de Fase 1 financian parcialmente la Fase 2. Ciclo virtuoso |
| **Modelo de costos compartidos** | Plataforma central compartida entre sectores → economías de escala |
| **Métricas de valor por programa** | Cada programa reporta su ROI al directorio trimestralmente. Si no alcanza → ajustar o cancelar |
| **Bonos verdes** | Proyectos de optimización energética pueden acceder a financiamiento verde a tasas preferenciales |
| **Monetización de datos** | En el mediano plazo, los datos anonimizados y agregados pueden generar ingresos (analytics-as-a-service a terceros) |

### Sostenibilidad Ambiental (ESG)

- Migración a cloud reduce consumo de centros de datos propios ineficientes
- Optimización de rutas logísticas con ML → menos km recorridos → menos emisiones
- Gemelos Digitales de plantas → reducción de pruebas físicas costosas en energía
- Reportes ESG automáticos con datos de la plataforma → sin estimaciones manuales

---

## P14. Criterios Go / No-Go por Iniciativa

| Iniciativa | Criterios Go | Criterios No-Go |
|---|---|---|
| **RAG Corporativo** | Knowledge base validada por área, LLM con tasa de alucinación < 3%, equipo de curación de contenido designado | Si el comité legal bloquea el uso de LLMs externos con datos internos |
| **Edge + IIoT** | Protocolo de seguridad OT aprobado, proveedores de hardware certificados, plan de rollback por site | Si no existe conectividad mínima en las plantas objetivo o si los sindicatos bloquean la instrumentación |
| **MLOps + IA Predictiva** | Datos de entrenamiento disponibles y de calidad, comité de ética de IA establecido, métricas de negocio definidas antes del entrenamiento | Si el dataset tiene menos de 12 meses de histórico o sesgos detectados sin plan de mitigación |
| **IA Generativa empresarial** | Casos de uso claros y medibles, supervisión humana definida, cumplimiento AI Act verificado | Si los casos de uso son de alto riesgo sin aprobación regulatoria |
| **Agentes IA autónomos** | Alcance limitado y supervisado, human-in-the-loop para decisiones críticas, rollback automático si el agente falla | Si el agente toma decisiones financieras, médicas o legales sin revisión humana |
| **XR empresarial** | ROI demostrado en piloto con grupo reducido, plan de desinfección (salud), soporte de RRHH para adopción | Si la tasa de adopción en piloto es < 40% después de 3 meses |
| **Gemelos Digitales** | IIoT de la planta ya desplegado y generando datos confiables, plataforma de simulación seleccionada | Si no hay datos IoT en tiempo real del activo físico que se va a geminar |
| **Blockchain** | Al menos 3 partes externas necesitan confiar entre sí, regulación permite, volumen < 10,000 transacciones/hora | Si los datos deben borrarse, si solo hay 1 parte o si la velocidad requerida es < 1 segundo |
| **Criptografía PQC** | Siempre GO — amenaza "harvest now decrypt later" es real hoy | No hay criterios No-Go. Es obligatorio para proteger datos sensibles a largo plazo |
| **Computación Cuántica (prod.)** | Ventaja cuántica demostrada en el caso de uso específico, costo-beneficio favorable vs clásico | Si los problemas a resolver tienen soluciones clásicas aceptables o si el hardware es demasiado costoso para el beneficio |

---

## P15. Mejor Estrategia de Transformación Digital

### La estrategia recomendada: "Platform-First, Sector-Deep"

**Principio central:** construir primero la plataforma horizontal que habilita a todos los sectores, luego profundizar con soluciones específicas por sector.

**Fase 1 — Plataforma horizontal (Programas 1, 2, 3):** RAG Corporativo + Edge/IIoT + MLOps dan a los 6 sectores una base tecnológica compartida. Cada sector la usa de inmediato para resolver sus problemas más urgentes.

**Fase 2 — Profundidad sectorial:** sobre esa base, cada sector desarrolla sus soluciones avanzadas: gemelos digitales de hospitales, agentes de crédito en banca, mantenimiento autónomo en manufactura.

**Por qué no Big Bang:** una transformación de 320M en 40 países no puede ejecutarse toda a la vez. El enfoque por fases permite aprender, corregir y reinvertir los ahorros de las fases tempranas.

**Por qué no un enfoque por sector:** si salud va primero y banca después, se construyen 6 plataformas diferentes con el mismo costo cada una. La plataforma compartida crea economías de escala.

```
ESTRATEGIA VISUAL:

PLATAFORMA HORIZONTAL (Fase 1 — compartida)
├── RAG Corporativo → todos los sectores tienen asistentes
├── Edge + IIoT → todos los sectores tienen datos en tiempo real
└── MLOps → todos los sectores tienen modelos en producción

        Sobre esa base:
PROFUNDIDAD SECTORIAL (Fase 2-4)
├── Banca: agentes de crédito, detección de fraude avanzada
├── Salud: gemelo digital hospitalario, diagnóstico con IA
├── Manufactura: mantenimiento autónomo, calidad con visión
├── Energía: grid cuántico, renovables optimizadas
├── Logística: optimización cuántica de rutas
└── Telecom: predicción de fallos de red, churn prevention
```

**Factores críticos de éxito:**
1. El C-Suite debe ser visible y comprometido — no puede ser solo un proyecto de TI
2. Los Quick Wins en los primeros 6 meses son esenciales para mantener el momentum
3. El talento interno debe crecer en paralelo a la tecnología — no depender de consultoras
4. La gobernanza de datos es tan importante como la tecnología — sin datos de calidad, la IA no funciona

---

## P16. Procesamiento de Datos de Sensores en Python — EXPLICADO

```python
"""
Procesamiento de datos de sensores industriales — GLOBAL FUTURE GROUP
Simula sensores de múltiples sectores: manufactura, energía, logística

Demuestra el flujo completo de datos IoT:
  1. Generación de datos por sensores (simulación realista)
  2. Ingesta y validación de calidad
  3. Procesamiento en stream (detección de anomalías, agregaciones)
  4. Enriquecimiento con contexto
  5. Salida hacia dashboard y sistema de alertas

EXPLICACIÓN DE CONCEPTOS:
  - Por qué los datos de sensores son ruidosos y cómo manejarlo
  - Qué es un pipeline de stream processing
  - Cómo funciona la detección de anomalías con ventana deslizante
  - Cómo se agregan datos para reducir tráfico hacia cloud
"""

import random
import time
import math
import statistics
from dataclasses import dataclass, field
from datetime import datetime, timedelta
from collections import deque, defaultdict
from typing import List, Dict, Optional, Tuple
from enum import Enum

# ─────────────────────────────────────────────────────────────────
# CONCEPTO 1: TIPOS DE SENSORES POR SECTOR
# Cada sector genera diferentes tipos de datos
# ─────────────────────────────────────────────────────────────────

class Sector(Enum):
    MANUFACTURA = "manufactura"
    ENERGIA = "energia"
    LOGISTICA = "logistica"
    SALUD = "salud"
    TELECOM = "telecom"

@dataclass
class ConfigSensor:
    """Define cómo se comporta un tipo de sensor."""
    nombre: str
    unidad: str
    valor_base: float           # Valor típico en operación normal
    ruido_std: float            # Desviación estándar del ruido (sensores no son perfectos)
    drift_por_hora: float       # Deriva gradual (sensor que se calienta, batería que baja)
    umbral_warning: float
    umbral_critical: float
    es_mayor_critico: bool      # True: crítico si SUPERA umbral | False: si está POR DEBAJO

# Configuración por tipo de sensor y sector
SENSORES_CONFIG = {
    # MANUFACTURA
    "temp_motor":       ConfigSensor("Temperatura Motor",    "°C",  65.0,  0.8,  0.02,  80.0,  95.0,  True),
    "vibracion":        ConfigSensor("Vibración",            "mm/s", 2.5,  0.3,  0.01,   4.5,   7.0,  True),
    "presion_hidraulic":ConfigSensor("Presión Hidráulica",  "bar",  150.0, 2.0,  0.05, 180.0, 200.0, True),
    "calidad_producto": ConfigSensor("Calidad Producto",     "%",    98.5,  0.4, -0.01,  95.0,  90.0, False),

    # ENERGÍA
    "voltaje_linea":    ConfigSensor("Voltaje Línea",        "V",   220.0,  1.5,  0.005, 228.0, 235.0, True),
    "frecuencia_red":   ConfigSensor("Frecuencia Red",       "Hz",   50.0,  0.05, 0.001,  50.3,  50.5, True),
    "temp_transformad": ConfigSensor("Temperatura Transform","°C",   55.0,  1.2,  0.03,  80.0,  95.0, True),

    # LOGÍSTICA
    "temp_cadena_frio": ConfigSensor("Temperatura Cadena Frío","°C",  2.0,  0.5,  0.02,   5.0,   8.0, True),
    "nivel_combustible":ConfigSensor("Nivel Combustible",    "%",    75.0,  0.2, -0.5,   20.0,  10.0, False),
    "velocidad_flota":  ConfigSensor("Velocidad Flota",      "km/h", 80.0,  3.0,  0.0,  120.0, 140.0, True),

    # SALUD
    "temp_ambiente_hosp":ConfigSensor("Temperatura Hospital","°C",   22.0,  0.3,  0.005, 26.0,  30.0, True),
    "humedad_quirofano": ConfigSensor("Humedad Quirófano",   "%HR",  50.0,  1.0,  0.01,  60.0,  70.0, True),

    # TELECOM
    "latencia_red":     ConfigSensor("Latencia de Red",      "ms",    5.0,  0.8,  0.005, 15.0,  30.0, True),
    "uso_cpu_nodo":     ConfigSensor("Uso CPU Nodo",         "%",    35.0,  5.0,  0.1,   80.0,  95.0, True),
}

# ─────────────────────────────────────────────────────────────────
# CONCEPTO 2: GENERADOR DE SENSORES REALISTAS
# Los sensores reales tienen ruido, deriva y ocasionalmente fallan
# ─────────────────────────────────────────────────────────────────

@dataclass
class LecturaSensor:
    sensor_id: str
    tipo: str
    sector: str
    planta_id: str
    valor: float
    valor_validado: float       # Después de filtrar outliers obvios
    timestamp: datetime
    es_valida: bool             # False si el sensor parece estar fallando
    metadata: dict = field(default_factory=dict)

class GeneradorSensor:
    """
    Simula un sensor físico con sus imperfecciones reales.

    EXPLICACIÓN: Los sensores reales no generan datos perfectos.
    Tienen:
    - Ruido: fluctuaciones aleatorias alrededor del valor real
    - Deriva (drift): el valor base cambia lentamente con el tiempo
    - Fallas: lecturas completamente incorrectas (sensor dañado)
    - Valores fuera de rango físico posible
    """
    def __init__(self, sensor_id: str, tipo: str, sector: Sector, planta_id: str,
                 tiene_falla_simulada: bool = False):
        self.sensor_id = sensor_id
        self.tipo = tipo
        self.sector = sector
        self.planta_id = planta_id
        self.config = SENSORES_CONFIG[tipo]
        self.valor_actual = self.config.valor_base + random.gauss(0, self.config.ruido_std * 2)
        self.contador_lecturas = 0
        self.tiene_falla = tiene_falla_simulada
        self.timestamp_inicio = datetime.now()

    def leer(self) -> LecturaSensor:
        self.contador_lecturas += 1
        ahora = datetime.now()

        # 1. Calcular deriva acumulada (el sensor "se cansa")
        horas_operando = self.contador_lecturas / 60.0
        deriva = self.config.drift_por_hora * horas_operando

        # 2. Agregar ruido gaussiano (el sensor nunca es perfecto)
        ruido = random.gauss(0, self.config.ruido_std)

        # 3. Valor "verdadero" del proceso
        valor_proceso = self.config.valor_base + deriva + ruido

        # 4. Simular falla del sensor (5% de probabilidad si tiene_falla=True)
        es_falla_puntual = self.tiene_falla and (random.random() < 0.10)

        if es_falla_puntual:
            # Sensor en falla: valor completamente fuera de rango
            valor_raw = valor_proceso * random.choice([0.1, 5.0, -1.0])
        else:
            valor_raw = valor_proceso

        # 5. Validar si la lectura es físicamente posible
        es_valida = self._validar_rango_fisico(valor_raw)
        valor_validado = valor_raw if es_valida else self.valor_actual  # Usar último válido

        if es_valida:
            self.valor_actual = valor_validado

        return LecturaSensor(
            sensor_id=self.sensor_id,
            tipo=self.tipo,
            sector=self.sector.value,
            planta_id=self.planta_id,
            valor=round(valor_raw, 3),
            valor_validado=round(valor_validado, 3),
            timestamp=ahora,
            es_valida=es_valida,
            metadata={
                "deriva_acumulada": round(deriva, 4),
                "lecturas_totales": self.contador_lecturas,
            }
        )

    def _validar_rango_fisico(self, valor: float) -> bool:
        """Filtra lecturas físicamente imposibles (temperatura de -500°C, etc.)."""
        config = self.config
        rango_max = config.valor_base * 3
        rango_min = config.valor_base * -0.5
        return rango_min <= valor <= rango_max

# ─────────────────────────────────────────────────────────────────
# CONCEPTO 3: STREAM PROCESSOR
# Procesa los datos en tiempo real, aplica transformaciones y detecta anomalías
# ─────────────────────────────────────────────────────────────────

@dataclass
class EventoProcesado:
    lectura: LecturaSensor
    promedio_ventana: float
    desv_ventana: float
    z_score: float
    anomalia_detectada: bool
    tipo_anomalia: str          # "umbral", "estadistica", "invalida", "ninguna"
    nivel_alerta: str           # "OK", "WARNING", "CRITICAL"

class StreamProcessor:
    """
    Procesa el stream de lecturas de sensores en tiempo real.

    EXPLICACIÓN: El Stream Processing analiza los datos mientras fluyen,
    sin almacenarlos todos primero. Usa ventanas deslizantes (sliding windows)
    para calcular estadísticas sobre las N lecturas más recientes.

    Equivalente en producción: Apache Flink o Apache Spark Streaming
    procesando mensajes de Apache Kafka.
    """
    def __init__(self, ventana_size: int = 20):
        """
        ventana_size: cuántas lecturas pasadas considerar para estadísticas.
        En producción: ventana de tiempo (ej. últimos 5 minutos).
        """
        self.ventana_size = ventana_size
        # Una ventana por sensor para calcular estadísticas locales
        self.ventanas: Dict[str, deque] = defaultdict(lambda: deque(maxlen=ventana_size))
        self.eventos_procesados = 0
        self.alertas_generadas = 0

    def procesar(self, lectura: LecturaSensor) -> EventoProcesado:
        """
        Procesa una lectura individual y determina si hay anomalía.

        Pasos:
        1. Si la lectura no es válida → alerta de sensor fallido
        2. Agregar a ventana deslizante
        3. Calcular estadísticas de la ventana
        4. Detectar anomalía por umbral de negocio
        5. Detectar anomalía estadística (z-score)
        """
        self.eventos_procesados += 1
        config = SENSORES_CONFIG[lectura.tipo]

        # Paso 1: Lectura inválida (sensor fallando)
        if not lectura.es_valida:
            self.alertas_generadas += 1
            return EventoProcesado(
                lectura=lectura,
                promedio_ventana=0, desv_ventana=0, z_score=0,
                anomalia_detectada=True,
                tipo_anomalia="invalida",
                nivel_alerta="CRITICAL"
            )

        # Paso 2: Agregar a ventana deslizante
        ventana_key = lectura.sensor_id
        self.ventanas[ventana_key].append(lectura.valor_validado)
        valores_ventana = list(self.ventanas[ventana_key])

        # Paso 3: Estadísticas de la ventana
        promedio = statistics.mean(valores_ventana) if valores_ventana else lectura.valor_validado
        desv = statistics.stdev(valores_ventana) if len(valores_ventana) > 1 else 0.0
        z_score = abs(lectura.valor_validado - promedio) / desv if desv > 0 else 0.0

        # Paso 4: Detección por umbral de negocio
        nivel_umbral = self._detectar_umbral(lectura.valor_validado, config)

        # Paso 5: Detección estadística (z-score)
        # Z-score > 3: el valor está a más de 3 desviaciones estándar de la media
        # Significa que es muy inusual (ocurre ~0.3% de las veces en distribución normal)
        anomalia_estadistica = len(valores_ventana) >= 5 and z_score > 3.0

        # Determinar nivel final
        if nivel_umbral == "CRITICAL":
            tipo = "umbral"
            nivel = "CRITICAL"
        elif nivel_umbral == "WARNING":
            tipo = "umbral"
            nivel = "WARNING"
        elif anomalia_estadistica:
            tipo = "estadistica"
            nivel = "WARNING"
        else:
            tipo = "ninguna"
            nivel = "OK"

        hay_anomalia = nivel != "OK"
        if hay_anomalia:
            self.alertas_generadas += 1

        return EventoProcesado(
            lectura=lectura,
            promedio_ventana=round(promedio, 3),
            desv_ventana=round(desv, 3),
            z_score=round(z_score, 2),
            anomalia_detectada=hay_anomalia,
            tipo_anomalia=tipo,
            nivel_alerta=nivel
        )

    def _detectar_umbral(self, valor: float, config: ConfigSensor) -> str:
        if config.es_mayor_critico:
            if valor >= config.umbral_critical:
                return "CRITICAL"
            elif valor >= config.umbral_warning:
                return "WARNING"
        else:
            if valor <= config.umbral_critical:
                return "CRITICAL"
            elif valor <= config.umbral_warning:
                return "WARNING"
        return "OK"

# ─────────────────────────────────────────────────────────────────
# CONCEPTO 4: AGREGADOR DE DATOS
# Reduce el volumen de datos antes de enviarlos al cloud
# ─────────────────────────────────────────────────────────────────

class AgregadorDatos:
    """
    Agrega datos antes de enviarlos al cloud para reducir costos y tráfico.

    EXPLICACIÓN: Un sensor puede generar 1 lectura por segundo.
    Con 10,000 sensores = 10,000 mensajes/segundo al cloud.
    El agregador calcula estadísticas por período (ej. cada 1 minuto)
    y solo envía 1 mensaje resumen por sensor → 99% menos tráfico.

    En producción: Apache Flink "tumbling windows" o "sliding windows".
    """
    def __init__(self, periodo_seg: int = 60):
        self.periodo_seg = periodo_seg
        self.buffers: Dict[str, List[float]] = defaultdict(list)
        self.ultimo_flush: Dict[str, datetime] = {}

    def agregar(self, lectura: LecturaSensor) -> Optional[dict]:
        """
        Acumula lecturas. Cuando pasa el período, devuelve el resumen.
        Retorna None mientras acumula, devuelve el resumen al hacer flush.
        """
        key = lectura.sensor_id
        if lectura.es_valida:
            self.buffers[key].append(lectura.valor_validado)

        ahora = datetime.now()
        ultimo = self.ultimo_flush.get(key)

        # Hacer flush si pasó el período o si es la primera vez
        if ultimo is None or (ahora - ultimo).seconds >= self.periodo_seg:
            if not self.buffers[key]:
                return None

            valores = self.buffers[key]
            resumen = {
                "sensor_id": key,
                "tipo": lectura.tipo,
                "sector": lectura.sector,
                "planta_id": lectura.planta_id,
                "periodo_inicio": (ultimo or ahora).isoformat(),
                "periodo_fin": ahora.isoformat(),
                "n_lecturas": len(valores),
                "promedio": round(statistics.mean(valores), 3),
                "minimo": round(min(valores), 3),
                "maximo": round(max(valores), 3),
                "desv_std": round(statistics.stdev(valores) if len(valores) > 1 else 0, 3),
            }

            self.buffers[key] = []
            self.ultimo_flush[key] = ahora
            return resumen

        return None

# ─────────────────────────────────────────────────────────────────
# CONCEPTO 5: PIPELINE COMPLETO DE PROCESAMIENTO
# ─────────────────────────────────────────────────────────────────

class PipelineSensores:
    """
    Pipeline completo: Sensor → Validación → Stream Processing → Agregación → Output

    FLUJO EN PRODUCCIÓN:
    Sensor físico
        │ MQTT
        ▼
    IoT Gateway (Azure IoT Hub / AWS IoT Core)
        │ Kafka topic por tipo de sensor
        ▼
    Apache Flink (stream processing en tiempo real)
        │
        ├──► Detección de anomalías → PagerDuty / alertas
        ├──► Agregaciones → Delta Lake (almacenamiento)
        └──► Features → Feature Store (para modelos ML)
    """
    def __init__(self, num_sensores: int = 30):
        self.processor = StreamProcessor(ventana_size=15)
        self.agregador = AgregadorDatos(periodo_seg=10)  # 10s para la demo
        self.sensores = self._crear_sensores(num_sensores)
        self.alertas_history: List[EventoProcesado] = []
        self.resumenes_cloud: List[dict] = []
        self.stats_globales = {
            "total_lecturas": 0,
            "lecturas_invalidas": 0,
            "anomalias_detectadas": 0,
            "resumenes_enviados_cloud": 0,
        }

    def _crear_sensores(self, n: int) -> List[GeneradorSensor]:
        """Crea una flota diversa de sensores en múltiples sectores."""
        tipos_por_sector = {
            Sector.MANUFACTURA: ["temp_motor", "vibracion", "presion_hidraulic", "calidad_producto"],
            Sector.ENERGIA:     ["voltaje_linea", "frecuencia_red", "temp_transformad"],
            Sector.LOGISTICA:   ["temp_cadena_frio", "nivel_combustible", "velocidad_flota"],
            Sector.SALUD:       ["temp_ambiente_hosp", "humedad_quirofano"],
            Sector.TELECOM:     ["latencia_red", "uso_cpu_nodo"],
        }

        sensores = []
        sectores = list(tipos_por_sector.keys())
        for i in range(n):
            sector = sectores[i % len(sectores)]
            tipos = tipos_por_sector[sector]
            tipo = tipos[i % len(tipos)]
            planta_id = f"PLANTA-{sector.value[:3].upper()}-{(i // len(sectores)) + 1:02d}"
            sensor_id = f"SEN-{sector.value[:3].upper()}-{tipo[:4].upper()}-{i+1:03d}"
            tiene_falla = (i % 9 == 0)  # ~11% de sensores con problemas
            sensores.append(GeneradorSensor(sensor_id, tipo, sector, planta_id, tiene_falla))

        return sensores

    def ejecutar_ciclo(self) -> Tuple[List[EventoProcesado], List[dict]]:
        """Procesa un ciclo de lecturas de todos los sensores."""
        eventos_ciclo = []
        resumenes_ciclo = []

        for sensor in self.sensores:
            # 1. Obtener lectura del sensor
            lectura = sensor.leer()
            self.stats_globales["total_lecturas"] += 1
            if not lectura.es_valida:
                self.stats_globales["lecturas_invalidas"] += 1

            # 2. Procesar en stream (detección de anomalías)
            evento = self.processor.procesar(lectura)
            eventos_ciclo.append(evento)

            if evento.anomalia_detectada:
                self.stats_globales["anomalias_detectadas"] += 1
                self.alertas_history.append(evento)

            # 3. Agregar para cloud (reducir tráfico)
            resumen = self.agregador.agregar(lectura)
            if resumen:
                resumenes_ciclo.append(resumen)
                self.resumenes_cloud.append(resumen)
                self.stats_globales["resumenes_enviados_cloud"] += 1

        return eventos_ciclo, resumenes_ciclo

    def mostrar_dashboard(self, ciclo: int, eventos: List[EventoProcesado],
                          resumenes: List[dict]):
        """Dashboard en consola del estado de la red de sensores."""
        ahora = datetime.now().strftime("%H:%M:%S")
        print(f"\n{'═'*70}")
        print(f"  PIPELINE SENSORES — GLOBAL FUTURE GROUP | Ciclo #{ciclo} | {ahora}")
        print(f"{'═'*70}")

        # Estado por sector
        print("  ESTADO POR SECTOR:")
        sector_stats = defaultdict(lambda: {"ok": 0, "warn": 0, "crit": 0, "inv": 0})
        for ev in eventos:
            sec = ev.lectura.sector
            if not ev.lectura.es_valida:
                sector_stats[sec]["inv"] += 1
            elif ev.nivel_alerta == "CRITICAL":
                sector_stats[sec]["crit"] += 1
            elif ev.nivel_alerta == "WARNING":
                sector_stats[sec]["warn"] += 1
            else:
                sector_stats[sec]["ok"] += 1

        for sector, stats in sorted(sector_stats.items()):
            total = sum(stats.values())
            barra = (
                "[" +
                "█" * stats["ok"] +
                "▒" * stats["warn"] +
                "░" * stats["crit"] +
                "X" * stats["inv"] +
                "]"
            )
            print(f"  {sector:<15} {barra:<20} "
                  f"OK={stats['ok']:2d} WARN={stats['warn']:2d} "
                  f"CRIT={stats['crit']:2d} INV={stats['inv']:2d}")

        # Alertas del ciclo
        alertas_ciclo = [e for e in eventos if e.anomalia_detectada]
        if alertas_ciclo:
            print(f"\n  ALERTAS ESTE CICLO ({len(alertas_ciclo)}):")
            for ev in alertas_ciclo[:6]:
                icono = "!!" if ev.nivel_alerta == "CRITICAL" else " !"
                print(f"  {icono} [{ev.nivel_alerta:<8}] {ev.lectura.sensor_id:<28} "
                      f"Val={ev.lectura.valor_validado:7.2f} {SENSORES_CONFIG[ev.lectura.tipo].unidad:<6} "
                      f"Z={ev.z_score:.1f} [{ev.tipo_anomalia}]")

        # Resúmenes enviados a cloud
        if resumenes:
            print(f"\n  AGREGACIONES ENVIADAS A CLOUD ({len(resumenes)}):")
            for r in resumenes[:4]:
                print(f"  → {r['sensor_id']:<30} Prom={r['promedio']:7.2f} "
                      f"[{r['minimo']:.2f} – {r['maximo']:.2f}] n={r['n_lecturas']}")

        # Stats globales
        s = self.stats_globales
        print(f"\n  TOTALES: Lecturas={s['total_lecturas']:,} | "
              f"Inválidas={s['lecturas_invalidas']:,} | "
              f"Anomalías={s['anomalias_detectadas']:,} | "
              f"Resúmenes cloud={s['resumenes_enviados_cloud']:,}")

        # Tasa de reducción de datos
        if s["total_lecturas"] > 0:
            reduccion = (1 - s["resumenes_enviados_cloud"] / s["total_lecturas"]) * 100
            print(f"  Reducción de datos hacia cloud: {reduccion:.1f}% "
                  f"(de {s['total_lecturas']:,} lecturas → {s['resumenes_enviados_cloud']:,} resúmenes)")

def main():
    print("=" * 70)
    print("  PIPELINE IoT INDUSTRIAL — GLOBAL FUTURE GROUP")
    print("  Manufactura | Energía | Logística | Salud | Telecom")
    print("=" * 70)

    NUM_SENSORES = 30
    NUM_CICLOS = 6
    PAUSA = 2

    pipeline = PipelineSensores(num_sensores=NUM_SENSORES)
    print(f"\n  Sensores inicializados: {NUM_SENSORES}")
    print(f"  Sectores: {len(set(s.sector for s in pipeline.sensores))}")

    for ciclo in range(1, NUM_CICLOS + 1):
        eventos, resumenes = pipeline.ejecutar_ciclo()
        pipeline.mostrar_dashboard(ciclo, eventos, resumenes)
        if ciclo < NUM_CICLOS:
            time.sleep(PAUSA)

    print(f"\n{'═'*70}")
    print("  RESUMEN FINAL")
    print(f"{'═'*70}")
    s = pipeline.stats_globales
    tasa_anomalia = s["anomalias_detectadas"] / s["total_lecturas"] * 100
    tasa_invalidas = s["lecturas_invalidas"] / s["total_lecturas"] * 100
    print(f"  Total lecturas procesadas : {s['total_lecturas']:,}")
    print(f"  Lecturas inválidas        : {s['lecturas_invalidas']:,} ({tasa_invalidas:.1f}%)")
    print(f"  Anomalías detectadas      : {s['anomalias_detectadas']:,} ({tasa_anomalia:.1f}%)")
    print(f"  Resúmenes enviados cloud  : {s['resumenes_enviados_cloud']:,}")
    print(f"\n  ESCALABILIDAD CON 40 PAÍSES:")
    print(f"  Si cada país tiene 750 sensores → {40*750:,} sensores globales")
    print(f"  A 1 lectura/seg → {40*750:,} mensajes/seg al pipeline")
    print(f"  Con agregación 60s → {40*750 // 60:,} resúmenes/seg al cloud")
    print(f"  Reducción de tráfico: ~98%")

if __name__ == "__main__":
    main()
```

### Explicación del pipeline — Conceptos clave

**¿Por qué los datos de sensores son "ruidosos"?**

Un sensor de temperatura no lee exactamente 65.0°C — lee 65.3°C, luego 64.8°C, luego 65.1°C. Esa variación es ruido natural del hardware. El pipeline debe distinguir entre ruido normal y una anomalía real. Para eso se usa la ventana deslizante: si el promedio de las últimas 20 lecturas sube, es una tendencia real; si solo sube 1 lectura y vuelve, es ruido.

**¿Qué es el z-score y por qué se usa?**

```
z-score = (valor_actual - promedio_ventana) / desviación_estándar_ventana

Si z > 3.0 → el valor está a más de 3 desviaciones del promedio
En distribución normal → esto ocurre solo el 0.3% del tiempo
→ Probablemente es una anomalía real, no ruido
```

**¿Por qué agregar datos antes de enviar al cloud?**

```
Sin agregación:
  30 sensores × 1 lectura/segundo = 30 mensajes/segundo
  × 86,400 segundos/día = 2,592,000 mensajes/día
  × costo cloud por mensaje = $$$ enorme

Con agregación cada 60 segundos:
  30 sensores × 1 resumen/minuto = 30 mensajes/minuto
  Reducción: 60x menos mensajes → 60x menos costo cloud

El truco: las anomalías se detectan localmente (en el edge) en tiempo real
y la alerta llega al cloud instantáneamente. Los datos normales se agregan.
```

**Equivalente en producción:**

| Componente Python | Equivalente en producción |
|---|---|
| `GeneradorSensor.leer()` | Sensor físico MQTT publicando en Azure IoT Hub |
| `StreamProcessor.procesar()` | Apache Flink leyendo de Kafka y procesando en tiempo real |
| `AgregadorDatos` | Flink tumbling window de 1 minuto → escritura en Delta Lake |
| `deque(maxlen=15)` | Flink sliding window de N eventos para estadísticas |
| `alertas_history` | PagerDuty / Grafana Alerting recibiendo eventos críticos |
| `resumenes_cloud` | Delta Lake en Azure Data Lake Storage / AWS S3 |

---

## Resumen Ejecutivo para el Directorio

**GLOBAL FUTURE GROUP debe apostar por tres programas en la Fase 1:**

1. **RAG Corporativo Global** — $35M — Payback 15 meses — El conocimiento corporativo de 40 países en manos de cada empleado
2. **Edge + IIoT Multi-sector** — $55M — Payback 11 meses — Datos en tiempo real de todas las operaciones físicas
3. **MLOps + IA Predictiva** — $40M — Payback 13 meses — Inteligencia operativa que multiplica el valor de los dos anteriores

**Total Fase 1: $130M | ROI 5 años: 419% | Payback promedio: 13 meses**

La estrategia **"Platform-First, Sector-Deep"** garantiza que la inversión crea valor para los 6 sectores simultáneamente desde el año 1, sin duplicar infraestructura. Los ahorros de la Fase 1 financian parcialmente las Fases 2, 3 y 4.

Las tecnologías excluidas en la Fase 1 (Blockchain, XR, Gemelos Digitales, Cuántico) no se descartan — se pilotean con presupuesto menor y se escalan cuando la plataforma base está madura y el hardware/regulación lo permiten.
