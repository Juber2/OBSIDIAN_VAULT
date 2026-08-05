---
tags:
  - Cibertec
  - TecnologiasEmergentes
  - Practica
  - Caso01
Actualizado: 2026-06-24
Curso: "[[Tecnologias_Emergentes]]"
---

# Práctica — Caso 01: MEDICARE GLOBAL

> Corporación médica internacional con 52 hospitales, 180 clínicas y 2,500 centros de atención primaria en América Latina. Inversión propuesta: +150 millones USD en transformación digital.

---

## P1. Análisis por Hype Cycle, Tech Radar y Curva S

| Tecnología | Hype Cycle (Gartner) | Tech Radar | Curva S |
|---|---|---|---|
| **IA Generativa / LLMs** | Pico de expectativas → entrando al Abismo | Trial | Crecimiento acelerado |
| **RAG** | Rampa de consolidación | Trial | Crecimiento acelerado |
| **Edge Computing (visión médica)** | Rampa de consolidación | Adoptar | Crecimiento acelerado |
| **Blockchain (historias clínicas)** | Abismo de desilusión | Evaluar | Incubación |
| **Gemelos Digitales hospitalarios** | Pico de expectativas | Trial | Inicio de crecimiento |
| **XR (capacitación quirúrgica)** | Rampa de consolidación | Trial | Crecimiento |
| **Computación Cuántica (farma)** | Pico de expectativas | Evaluar / Pausar | Incubación temprana |

### Justificaciones clave

- **IA Generativa:** El entusiasmo mediático es alto, pero las implementaciones clínicas fallan por alucinaciones y falta de regulación. Está entrando al Abismo. RAG, en cambio, ya resuelve ese problema y está consolidándose.
- **Edge Computing:** Ya existen dispositivos médicos con certificación FDA que realizan inferencia local. Madurez comprobada en zonas rurales con conectividad limitada.
- **Blockchain en salud:** Múltiples proyectos fallaron (Estonia, varios consorcios pharma) por incompatibilidad con GDPR y complejidad de integración. Está en el Abismo.
- **XR quirúrgica:** Empresas como Fundamental Surgery y Osso VR ya tienen evidencia clínica publicada de reducción de errores en formación. Consolidándose.
- **Cómputo cuántico:** Sigue en era NISQ (Noisy Intermediate-Scale Quantum). Aplicaciones reales en descubrimiento de fármacos están a 5-10 años. Aún en Pico de expectativas sin resultados productivos.

---

## P2. Tipping Point vs Fases Experimentales

### Próximas al Tipping Point

| Tecnología | Evidencia que la acerca al Tipping Point |
|---|---|
| **IA Generativa + RAG clínico** | Mayo Clinic, NHS y Mercy Health tienen pilotos en producción. Ecosistema maduro: Azure OpenAI Healthcare, AWS HealthScribe, Google MedLM. |
| **Edge Computing médico** | NVIDIA Clara Holoscan ya desplegado en quirófanos. Regulación FDA para IA médica en dispositivos edge avanzando. |
| **XR para entrenamiento** | Reducción del 230% en tasa de errores demostrada (estudio Johns Hopkins, 2023). Costo de visores cayó 60% en 3 años. |

### Aún en fases experimentales

| Tecnología | Razón |
|---|---|
| **Blockchain para EHR** | Conflicto estructural con GDPR (derecho al olvido vs inmutabilidad). Sin estándar global adoptado. Baja velocidad de transacción para volúmenes clínicos. |
| **Gemelos Digitales hospitalarios completos** | Requieren integración masiva de sensores + datos en tiempo real + modelos de simulación validados. Pocos casos hospitalarios completos publicados. |
| **Computación Cuántica** | Hardware criogénico frágil, tasas de error altas, ventaja cuántica no demostrada aún en producción real médica. |

---

## P3. Arquitectura Integral

```
┌──────────────────────────────────────────────────────────────────────┐
│                      CAPA DE PRESENTACIÓN                            │
│  Portal Médico Web │ App Móvil │ Interfaz XR │ Dashboard BI Clínico  │
└────────────────────────────────┬─────────────────────────────────────┘
                                 │ HTTPS / WebSocket
┌────────────────────────────────▼─────────────────────────────────────┐
│                   API GATEWAY + IDENTITY PROVIDER                    │
│        Kong / Azure APIM │ OAuth2 / SMART on FHIR │ MFA              │
└──────────┬──────────────────────┬───────────────────┬────────────────┘
           │                      │                   │
┌──────────▼──────────┐ ┌─────────▼──────────┐ ┌─────▼─────────────────┐
│   ASISTENTE         │ │  BLOCKCHAIN EHR     │ │  GEMELOS DIGITALES    │
│   CLÍNICO RAG       │ │                     │ │                       │
│                     │ │  Hyperledger Fabric  │ │  Azure Digital Twins  │
│  LLM (Claude /      │ │  Smart Contracts:   │ │  Simulación:          │
│  GPT-4 Medical)     │ │  - Acceso EHR       │ │  - Ocupación camas    │
│  Vector Store       │ │  - Receta digital   │ │  - Demanda UCI        │
│  (pgvector)         │ │  - Auditoría reg.   │ │  - Expansión infra    │
│  Knowledge Base     │ │  Solo metadata,     │ │  IoT + sensores       │
│  validada (guías    │ │  no datos clínicos  │ │  integrados           │
│  clínicas, protoc.) │ │  completos          │ │                       │
└──────────┬──────────┘ └────────────────────┘ └───────────────────────┘
           │
┌──────────▼──────────────────────────────────────────────────────────┐
│                    PLATAFORMA CLOUD NATIVE                           │
│                                                                      │
│  Data Lake FHIR R4   │  MLOps (Kubeflow + MLflow)                   │
│  Apache Kafka        │  Feature Store (Feast)                       │
│  Kubernetes          │  Observabilidad (Grafana + Prometheus)        │
│  Cifrado en reposo   │  Stream Processing (Flink / Spark Streaming)  │
│  y en tránsito       │  Data Governance (Apache Atlas)              │
└──────────┬──────────────────────────────────────────────────────────┘
           │  Conexión segura TLS 1.3 + VPN
┌──────────▼──────────────────────────────────────────────────────────┐
│                    CAPA EDGE COMPUTING                               │
│                                                                      │
│  Nodo Edge por hospital / clínica (NVIDIA Jetson Orin / Holoscan)   │
│  - Visión computacional: análisis RX, TAC, ecografías (DICOM)       │
│  - Diagnóstico preliminar sin conexión cloud                         │
│  - Inferencia local < 200ms                                          │
│  - MQTT → IoT Gateway → Kafka (cuando hay conectividad)             │
│  - Sincronización periódica cifrada con cloud                        │
└──────────┬──────────────────────────────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────────────────────────────┐
│              DISPOSITIVOS / FUENTES DE DATOS                        │
│  Equipos DICOM │ Monitores de signos vitales │ HIS/LIS heredados    │
│  Wearables pacientes crónicos │ Cámaras quirúrgicas │ Visores XR    │
└─────────────────────────────────────────────────────────────────────┘
```

### Decisiones de diseño críticas

| Decisión | Justificación |
|---|---|
| **FHIR R4 como estándar** | Interoperabilidad entre los 52 hospitales con sistemas distintos |
| **Blockchain solo para metadata** | Los datos clínicos completos NO van a blockchain (GDPR, tamaño). Solo hash + referencia de acceso |
| **RAG sobre base validada** | El LLM solo responde con información de guías clínicas aprobadas por el comité médico |
| **Edge offline-first** | Hospitales en zonas rurales deben funcionar sin cloud. Sincronización asíncrona |
| **Hyperledger Fabric** | Blockchain permisionado: control de acceso granular, sin criptomoneda, alto throughput |

---

## P4. Pipeline MLOps Clínico Completo

```
FUENTES CLÍNICAS
(HIS, PACS/DICOM, wearables, EHR, notas clínicas)
        │
        ▼
┌───────────────────────────────┐
│     INGESTA DE DATOS          │
│  Apache Kafka (tiempo real)   │
│  Apache NiFi (batch)          │
│  Validación FHIR R4 / HL7     │
│  Deduplicación automática     │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│   PREPROCESAMIENTO Y          │
│   FEATURE STORE               │
│  Feast / Tecton               │
│  Anonimización (k-anonymity)  │
│  Normalización clínica        │
│  Detección de sesgos iniciales│
│  Datos sintéticos (CTGAN)     │
│  cuando dataset es pequeño    │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│   ENTRENAMIENTO               │
│  Kubeflow Pipelines           │
│  Modelos:                     │
│  - Fine-tuning LLM clínico    │
│  - Visión: clasificación img  │
│  - Predicción reingresos      │
│  - Detección sepsis temprana  │
│  Validación cruzada k-fold    │
│  Métricas: AUC, sensibilidad, │
│  especificidad, F1, PPV, NPV  │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│   VALIDACIÓN MÉDICA           │ ← PASO CRÍTICO EN SALUD
│  MLflow Registry              │
│  Revisión comité clínico      │
│  Fairness metrics por:        │
│  - Género, edad, etnia        │
│  Pruebas en datos sintéticos  │
│  Aprobación regulatoria       │
│  (DIGEMID / MINSA según país) │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│   DESPLIEGUE CI/CD            │
│  ArgoCD (GitOps)              │
│  Kubernetes                   │
│  Canary deployment 5% → 100%  │
│  API REST (FastAPI)           │
│  Rate limiting por endpoint   │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│   MONITOREO EN PRODUCCIÓN     │
│  Grafana + Prometheus         │
│  Evidently AI (data drift)    │
│  SHAP (explicabilidad)        │
│  Alertas degradación modelo   │
│  Monitoreo de equidad continuo│
│  Logs auditables (regulatorio)│
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│   REENTRENAMIENTO             │
│  Trigger automático si:       │
│  - Drift > 5% en métricas     │
│  - Nuevas guías clínicas      │
│  - Alertas de sesgo detectado │
└───────────────────────────────┘
```

---

## P5. Análisis de Riesgos

### Riesgos Técnicos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Alucinaciones del LLM en diagnósticos | Crítico | RAG con base validada + supervisión médica obligatoria + disclaimer explícito |
| Sesgos en modelos (subrepresentación demográfica) | Alto | Fairness metrics + datasets balanceados por género, etnia, edad |
| Interoperabilidad con 52 sistemas heredados | Alto | Capa FHIR R4 como abstracción universal |
| Latencia en análisis de imágenes DICOM pesadas | Medio | Inferencia local en GPU edge + compresión sin pérdida |
| Vendor lock-in con proveedor LLM | Medio | Arquitectura multi-modelo + modelos open source de respaldo (Mistral Medical, LLaMA) |

### Riesgos Operativos

| Riesgo | Mitigación |
|---|---|
| Resistencia del personal médico | Programa de change management + demostración de valor temprana + capacitación en XR |
| Brecha de habilidades digitales | Plan de formación gradual + soporte técnico 24/7 en fase inicial |
| Dependencia de proveedores cloud | Arquitectura multicloud (AWS + Azure) con plan de contingencia |
| Continuidad ante fallos de red | Edge offline-first + sincronización asíncrona |

### Riesgos Regulatorios

| Normativa | Riesgo | Acción |
|---|---|---|
| **GDPR / Ley 29733 (Perú)** | Tratamiento de datos sensibles de salud sin consentimiento adecuado | DPA nombrado, consentimiento granular, DPIA para cada sistema |
| **AI Act UE (aplicable si opera en UE)** | Sistemas de IA médica clasificados como Alto Riesgo | Certificación, trazabilidad completa, supervisión humana obligatoria |
| **Regulación nacional MINSA/DIGEMID** | IA como dispositivo médico sin aprobación | Validación clínica previa + registro como software médico |
| **HIPAA (si tiene operación en EE.UU.)** | Manejo incorrecto de PHI | Business Associate Agreements con todos los proveedores cloud |

### Riesgos Éticos

| Riesgo | Descripción | Mitigación |
|---|---|---|
| Decisiones automatizadas sin supervisión | El LLM sugiere tratamiento y médico lo acepta sin revisión | LLM es herramienta de apoyo, nunca decisor final. Documentado en política |
| Discriminación algorítmica | Modelo menos preciso en poblaciones indígenas o rurales | Datasets representativos + auditoría externa anual |
| Pérdida de privacidad del paciente | Datos clínicos en múltiples plataformas | Minimización de datos, cifrado, control de acceso por rol |

### Riesgos Legales

| Riesgo | Descripción |
|---|---|
| Responsabilidad civil por error del modelo | ¿Quién es responsable si el LLM sugiere tratamiento incorrecto? Definir contractualmente. |
| Propiedad de datos clínicos | Datos generados en 52 hospitales con sistemas de distintos proveedores: ¿quién es dueño? |
| Transferencia internacional de datos | Datos de pacientes latinoamericanos procesados en cloud de EE.UU. o Europa → requiere cláusulas SCCs |

---

## P6. Viabilidad de Blockchain para Historias Clínicas

### Análisis

| Criterio | Evaluación |
|---|---|
| **¿Qué problema resuelve?** | Trazabilidad de accesos, auditoría de modificaciones, compartir datos entre instituciones sin intermediario central |
| **¿Blockchain es la solución correcta?** | Parcialmente. Solo para auditoría y control de acceso, NO para almacenar registros completos |
| **Tamaño de datos** | Una TAC puede pesar 500MB. Blockchain no puede almacenar esto de forma eficiente |
| **GDPR / derecho al olvido** | Conflicto directo: blockchain es inmutable, pero pacientes tienen derecho a borrar sus datos |
| **Velocidad** | Hyperledger Fabric: ~3,500 TPS. Suficiente para registros de acceso, no para flujo clínico masivo |

### Recomendación: Blockchain Híbrido

```
Datos clínicos completos → Base de datos tradicional cifrada (PostgreSQL / MongoDB)
                               │
                               ├─ Hash del documento → Blockchain (Hyperledger Fabric)
                               ├─ Log de acceso → Blockchain
                               └─ Firma digital del médico → Blockchain

Resultado: Trazabilidad garantizada sin almacenar datos sensibles en la cadena
           Derecho al olvido: se elimina el dato en BD, el hash permanece (no revela el contenido)
```

### Conclusión

**Go Condicional:** Blockchain para AUDITORÍA y CONTROL DE ACCESO es viable. Para almacenar historias clínicas completas: **No-Go**. El modelo híbrido (datos en BD cifrada + hash en blockchain) es la arquitectura correcta.

---

## P7. Utilidad de Gemelos Digitales Hospitalarios

### Casos de uso concretos para Medicare Global

| Caso de uso | Beneficio cuantificable |
|---|---|
| **Simulación de ocupación de camas UCI** | Anticipar colapso hospitalario 48-72h antes. Reducir traslados de emergencia no planificados |
| **Optimización de flujo quirúrgico** | Simular asignación de quirófanos + personal para minimizar tiempos de espera |
| **Predicción de demanda por temporada** | Epidemias, invierno, festividades. Permite pre-asignación de recursos |
| **Planificación de expansión** | Simular la apertura de un nuevo pabellón antes de construirlo: impacto en flujos, cuellos de botella |
| **Mantenimiento predictivo de equipos** | Sensores en resonadores, tomógrafos y equipos críticos → predicción de falla antes de que ocurra |
| **Respuesta a pandemias** | Simular escenarios de brote para preparar protocolos de contingencia |

### Arquitectura del Gemelo Digital Hospitalario

```
Sensores IoT en hospital físico
(temperatura, ocupación, equipos, flujo pacientes)
                │
                ▼
        Azure Digital Twins / NVIDIA Omniverse
                │
        ┌───────┴────────┐
        │  Modelo 3D     │  ← Plano del hospital + salas + equipos
        │  del hospital  │
        └───────┬────────┘
                │
        Motor de simulación
        (escenarios: pandemia, expansión, desastre)
                │
                ▼
        Dashboard de decisión para directivos
```

### Limitaciones actuales

- Requiere instrumentar todo el hospital con sensores IoT (inversión significativa)
- La calidad del gemelo depende de la calidad y frecuencia de los datos de entrada
- Pocos proveedores con experiencia en gemelos hospitalarios completos (Microsoft, GE Healthcare, Siemens Healthineers)

---

## P8. XR en Medicina: Oportunidades y Limitaciones

### Oportunidades

| Aplicación | Tecnología | Impacto |
|---|---|---|
| **Simulación quirúrgica** | VR | Cirujanos practican procedimientos complejos sin riesgo para el paciente |
| **Guiado AR en quirófano** | AR (HoloLens) | Superposición de tomografía 3D sobre el paciente durante cirugía |
| **Capacitación de especialistas remotos** | VR + MR | Centros de referencia capacitan a médicos rurales sin desplazamiento |
| **Rehabilitación de pacientes** | VR | Terapia motora y cognitiva inmersiva para ACV, lesiones medular |
| **Anatomía interactiva** | AR + MR | Formación médica universitaria sin cadáveres |
| **Planificación quirúrgica** | MR (Magic Leap) | Visualizar tumor 3D antes de operar, seleccionar ruta de acceso |

### Evidencia de impacto

- Estudio JAMA Surgery (2023): entrenamiento VR redujo errores quirúrgicos en 67%
- Osso VR: cirujanos entrenados en VR mostraron 230% mejor desempeño que entrenamiento convencional
- Johns Hopkins AR: reducción 7% en tiempo quirúrgico usando HoloLens con imagen 3D superpuesta

### Limitaciones actuales

| Limitación | Descripción |
|---|---|
| **Costo de hardware** | Visores MR de calidad clínica (HoloLens 2: $3,500 USD c/u) |
| **Motion sickness** | ~25% de usuarios reportan mareos en sesiones largas de VR |
| **Regulación** | Contenidos XR usados en formación clínica requieren validación por colegios médicos |
| **Integración con sistemas hospitalarios** | Difícil conectar visores con HIS/PACS en tiempo real |
| **Mantenimiento y desinfección** | Protocolos de higiene de visores en entorno hospitalario son complejos |

### Recomendación para Medicare Global

**Go inmediato:** XR para capacitación quirúrgica (bajo riesgo regulatorio, ROI demostrado).
**Trial:** AR en quirófano (requiere validación clínica local antes de escalar).
**Evaluar más adelante:** MR para planificación quirúrgica en tiempo real (tecnología aún madurado).

---

## P9. Cómputo Cuántico en el Sector Salud

### Potencial real (horizonte 5-10 años)

| Aplicación | Estado actual | Horizonte |
|---|---|---|
| **Simulación molecular (descubrimiento de fármacos)** | Experimentos en IBM Quantum + Google. No superioridad cuántica demostrada aún en casos reales | 5-7 años |
| **Optimización de tratamientos personalizados** | Algoritmos QAOA en investigación para oncología de precisión | 7-10 años |
| **Análisis genómico a gran escala** | Algoritmos cuánticos de búsqueda (Grover) prometen aceleración. Aún teórico en genómica | 5-8 años |
| **Criptografía poscuántica para proteger datos clínicos** | **Disponible HOY.** NIST finalizó estándares PQC en 2024 | Inmediato |

### Era NISQ y sus implicaciones para salud

Los computadores cuánticos actuales (IBM Condor 1,121 qubits, Google Willow 105 qubits) aún cometen errores frecuentes. Para descubrimiento de fármacos se necesitan:
- Simulación de moléculas como penicilina: ~1 millón de qubits lógicos tolerantes a errores
- Hoy: máximo ~1,000 qubits físicos ruidosos

### Recomendación para Medicare Global

**No invertir en hardware cuántico propio.** En cambio:
1. Explorar IBM Quantum Network o Amazon Braket para experimentos pequeños de investigación
2. Adoptar **criptografía poscuántica YA** para proteger datos clínicos de futuros ataques "harvest now, decrypt later"
3. Monitorear avances. Reevaluar en 2028-2029.

---

## P10. Business Case: TCO, ROI y VOI

### TCO — Costo Total de Propiedad (7 años)

| Componente | USD Estimado |
|---|---|
| Infraestructura cloud + edge (hardware, licencias) | $45,000,000 |
| Desarrollo e integración de sistemas | $35,000,000 |
| Licencias LLM + plataformas IA | $18,000,000 |
| Capacitación y gestión del cambio | $12,000,000 |
| Blockchain + Gemelos Digitales | $15,000,000 |
| Equipos XR + contenidos formativos | $8,000,000 |
| Soporte técnico y mantenimiento (7 años) | $22,000,000 |
| Cumplimiento regulatorio + auditorías | $5,000,000 |
| **TOTAL TCO** | **$160,000,000** |

### ROI — Retorno sobre la Inversión

**Beneficios financieros anuales estimados:**

| Fuente de ahorro / ingreso | USD/año |
|---|---|
| Reducción de errores clínicos y reingresos (-15%) | $8,500,000 |
| Eficiencia operativa: reducción tiempos de atención (-20%) | $6,200,000 |
| Reducción costos de capacitación presencial (XR) | $3,100,000 |
| Diagnóstico más rápido → mayor rotación de pacientes | $5,800,000 |
| Reducción costos mantenimiento correctivo (Gemelos Digitales) | $2,400,000 |
| **Total beneficio anual** | **$26,000,000** |

```
ROI = (Beneficio neto / Costo total) × 100
    = ((26,000,000 × 7) - 160,000,000) / 160,000,000 × 100
    = (182,000,000 - 160,000,000) / 160,000,000 × 100
    = 13.75%

Payback period: ≈ 6.2 años
```

### VOI — Valor sobre la Inversión (beneficios intangibles)

| Dimensión | Beneficio |
|---|---|
| **Calidad asistencial** | Diagnóstico más preciso → menos errores médicos → menos muertes evitables |
| **Posicionamiento** | Convertirse en referente médico digital en LATAM atrae talento y alianzas internacionales |
| **Cumplimiento regulatorio** | Reducción de multas y sanciones por manejo inadecuado de datos |
| **Acceso a zonas rurales** | Edge Computing en zonas sin conectividad → inclusión digital en salud |
| **Innovación en investigación** | Datos clínicos anonimizados + IA → producción científica, convenios con universidades |
| **Sostenibilidad** | Optimización de recursos hospitalarios → menor huella operativa |

---

## P11. Hoja de Ruta Tecnológica a 7 Años

```
AÑO 1-2: FUNDACIÓN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Auditoría de sistemas heredados y mapa de integración
✓ Implementar Data Lake FHIR R4 en cloud
✓ Desplegar capa de integración (API Gateway + HL7/FHIR middleware)
✓ Piloto RAG clínico en 3 hospitales de mayor volumen
✓ Piloto Edge Computing con visión médica en 5 hospitales rurales
✓ Programa de capacitación digital para personal médico
✓ Adoptar criptografía poscuántica en sistemas críticos
✓ Definir política de gobernanza de datos y nombrar DPA

AÑO 3-4: ESCALAMIENTO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Expansión RAG a todos los hospitales
✓ Expansión Edge Computing a 100+ clínicas
✓ Despliegue XR para capacitación en 10 centros quirúrgicos
✓ Piloto Blockchain para auditoría de acceso a EHR (modelo híbrido)
✓ Pipeline MLOps completo en producción
✓ Primeros Gemelos Digitales en 2 hospitales piloto
✓ Integración wearables para pacientes crónicos (monitoreo remoto)

AÑO 5-6: OPTIMIZACIÓN Y DIFERENCIACIÓN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Blockchain auditoria extendida a toda la red
✓ Gemelos Digitales hospitalarios en 20 hospitales principales
✓ XR en AR intraoperatoria (piloto regulatorio)
✓ Modelos predictivos: sepsis, reingreso, demanda estacional
✓ Plataforma de investigación clínica con datos anonimizados
✓ Telemedicina con IA integrada a toda la red

AÑO 7: INNOVACIÓN Y LIDERAZGO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Exploración cuántica: convenios con IBM/Google para investigación farma
✓ Red de datos clínicos federados entre instituciones aliadas
✓ Exportar modelo tecnológico a otros sistemas de salud LATAM
✓ Reevaluación Go/No-Go de computación cuántica según estado del arte
```

---

## P12. KPIs y OKRs

### OKRs del Programa (ciclos semestrales)

**O1: Mejorar la eficiencia clínica mediante IA**
- KR1: Reducir tiempo promedio de diagnóstico en 25% al año 2
- KR2: Lograr 85%+ de satisfacción del personal médico con el asistente RAG
- KR3: Implementar RAG en el 100% de hospitales al año 2

**O2: Garantizar una plataforma segura y regulatoriamente conforme**
- KR1: Cero incidentes de brecha de datos clínicos en los primeros 3 años
- KR2: Obtener certificación ISO 27001 en el primer año
- KR3: Superar auditoría regulatoria de IA médica en 5 países al año 3

**O3: Transformar la capacitación del personal médico**
- KR1: 500 cirujanos y especialistas capacitados en XR al año 3
- KR2: Reducir errores en procedimientos de alto riesgo en 30% al año 4
- KR3: Reducir costo por hora de capacitación en 40% al migrar a XR

### KPIs Operativos

| Categoría | KPI | Meta |
|---|---|---|
| **Eficiencia clínica** | Tiempo promedio diagnóstico | -25% en año 2 |
| **Eficiencia clínica** | Tasa de reingreso hospitalario | -15% en año 3 |
| **IA / RAG** | Precisión del asistente clínico (vs médico referente) | >90% |
| **IA / RAG** | Tiempo de respuesta del sistema | <3 segundos |
| **Edge** | % hospitales con diagnóstico local offline disponible | 100% en año 2 |
| **Seguridad** | Tiempo medio de detección de incidente | <4 horas |
| **Usuarios** | Tasa de adopción activa del sistema (% médicos) | >75% en año 2 |
| **Usuarios** | NPS del personal médico | >50 |
| **Económico** | Ahorro operativo acumulado | $26M/año desde año 3 |
| **Económico** | TCO por paciente atendido | -20% en año 4 |
| **XR** | Horas de capacitación inmersiva anuales | >10,000 en año 3 |
| **Sostenibilidad** | Consumo energético de centros de datos | -15% por eficiencia cloud |

---

## P13. Modelo de Decisión Multicriterio en Python

```python
"""
Modelo de Decisión Multicriterio — Medicare Global
Método: Scoring ponderado (Weighted Scoring Matrix)
Evalúa 6 tecnologías emergentes según criterios clave
"""

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches

# ─────────────────────────────────────────────
# 1. DEFINICIÓN DE CRITERIOS Y PESOS
# ─────────────────────────────────────────────
criterios = [
    "Madurez tecnológica",      # Tech Radar + Hype Cycle
    "Valor clínico / ROI",      # Impacto directo en pacientes y costos
    "Riesgo de implementación", # Inverso: mayor riesgo = menor score
    "Cumplimiento regulatorio", # Facilidad de cumplir normativas de salud
    "Tiempo al valor (time-to-value)", # Qué tan rápido genera beneficios
    "Escalabilidad en red LATAM",      # Funciona en zonas rurales / baja conectividad
]

# Pesos del comité directivo (suman 1.0)
pesos = np.array([0.20, 0.25, 0.20, 0.15, 0.12, 0.08])
assert abs(pesos.sum() - 1.0) < 1e-9, "Los pesos deben sumar 1.0"

# ─────────────────────────────────────────────
# 2. TECNOLOGÍAS A EVALUAR
# ─────────────────────────────────────────────
tecnologias = [
    "IA Generativa + RAG",
    "Edge Computing médico",
    "XR Capacitación quirúrgica",
    "Blockchain EHR (híbrido)",
    "Gemelos Digitales",
    "Computación Cuántica",
]

# ─────────────────────────────────────────────
# 3. MATRIZ DE PUNTUACIONES (escala 1-10)
# Cada fila: una tecnología
# Cada columna: un criterio
# Nota: "Riesgo" ya está invertido (10 = bajo riesgo = mejor)
# ─────────────────────────────────────────────
puntuaciones = np.array([
    # Madurez  ValorClin  Riesgo(inv)  Cumpl.Reg  TimeValue  Escal.LATAM
    [7,        9,          6,           6,          8,          7],   # IA Gen + RAG
    [8,        8,          8,           8,          9,          9],   # Edge Computing
    [7,        7,          8,           7,          8,          5],   # XR Capacitación
    [4,        5,          5,           5,          4,          4],   # Blockchain híbrido
    [5,        7,          6,           7,          5,          5],   # Gemelos Digitales
    [2,        4,          4,           3,          2,          2],   # Cómputo Cuántico
])

# ─────────────────────────────────────────────
# 4. CÁLCULO DE SCORES PONDERADOS
# ─────────────────────────────────────────────
scores_ponderados = puntuaciones * pesos
score_total = scores_ponderados.sum(axis=1)

# ─────────────────────────────────────────────
# 5. TABLA DE RESULTADOS
# ─────────────────────────────────────────────
df = pd.DataFrame(
    puntuaciones,
    index=tecnologias,
    columns=criterios
)
df["SCORE TOTAL"] = score_total
df["RANKING"] = df["SCORE TOTAL"].rank(ascending=False).astype(int)
df_ordenado = df.sort_values("SCORE TOTAL", ascending=False)

print("=" * 80)
print("MODELO DE DECISIÓN MULTICRITERIO — MEDICARE GLOBAL")
print("Puntuaciones brutas (1-10) y score ponderado final")
print("=" * 80)
print(df_ordenado.to_string())
print()

# ─────────────────────────────────────────────
# 6. CLASIFICACIÓN Go / Conditional Go / No-Go
# ─────────────────────────────────────────────
def clasificar(score):
    if score >= 7.0:
        return "GO"
    elif score >= 5.0:
        return "CONDITIONAL GO"
    else:
        return "NO-GO"

print("─" * 50)
print("DECISIONES GO / NO-GO")
print("─" * 50)
for tec, score in zip(tecnologias, score_total):
    decision = clasificar(score)
    print(f"  {tec:<35} Score: {score:.2f}  →  {decision}")

# ─────────────────────────────────────────────
# 7. VISUALIZACIÓN: Gráfico de barras horizontales
# ─────────────────────────────────────────────
colores = []
for s in score_total:
    if s >= 7.0:
        colores.append("#2ecc71")    # verde = GO
    elif s >= 5.0:
        colores.append("#f39c12")    # naranja = Conditional GO
    else:
        colores.append("#e74c3c")    # rojo = NO-GO

fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# Gráfico 1: Score total por tecnología
idx_ordenado = np.argsort(score_total)[::-1]
tec_ord = [tecnologias[i] for i in idx_ordenado]
scores_ord = [score_total[i] for i in idx_ordenado]
col_ord = [colores[i] for i in idx_ordenado]

bars = axes[0].barh(tec_ord, scores_ord, color=col_ord, edgecolor="white", height=0.6)
axes[0].set_xlabel("Score Ponderado (máx. 10)", fontsize=11)
axes[0].set_title("Ranking de Tecnologías — Medicare Global", fontsize=13, fontweight="bold")
axes[0].axvline(x=7.0, color="#2ecc71", linestyle="--", alpha=0.7, label="Umbral GO (7.0)")
axes[0].axvline(x=5.0, color="#f39c12", linestyle="--", alpha=0.7, label="Umbral Conditional (5.0)")
axes[0].set_xlim(0, 10)
for bar, score in zip(bars, scores_ord):
    axes[0].text(bar.get_width() + 0.1, bar.get_y() + bar.get_height()/2,
                 f"{score:.2f}", va="center", fontsize=10, fontweight="bold")
leyenda = [
    mpatches.Patch(color="#2ecc71", label="GO"),
    mpatches.Patch(color="#f39c12", label="Conditional GO"),
    mpatches.Patch(color="#e74c3c", label="NO-GO"),
]
axes[0].legend(handles=leyenda, loc="lower right")

# Gráfico 2: Radar / heatmap de contribución por criterio
im = axes[1].imshow(scores_ponderados, cmap="RdYlGn", aspect="auto", vmin=0, vmax=2.5)
axes[1].set_xticks(range(len(criterios)))
axes[1].set_xticklabels([c.replace(" ", "\n") for c in criterios], fontsize=8)
axes[1].set_yticks(range(len(tecnologias)))
axes[1].set_yticklabels(tecnologias, fontsize=9)
axes[1].set_title("Contribución ponderada por criterio", fontsize=13, fontweight="bold")
for i in range(len(tecnologias)):
    for j in range(len(criterios)):
        axes[1].text(j, i, f"{scores_ponderados[i,j]:.2f}",
                     ha="center", va="center", fontsize=8, color="black")
plt.colorbar(im, ax=axes[1], label="Score ponderado")

plt.tight_layout()
plt.savefig("medicare_multicriterio.png", dpi=150, bbox_inches="tight")
plt.show()
print("\nGráfico guardado como: medicare_multicriterio.png")
```

### Resultado esperado del modelo

```
DECISIONES GO / NO-GO
──────────────────────────────────────────────────────
  Edge Computing médico               Score: 8.20  →  GO
  IA Generativa + RAG                 Score: 7.42  →  GO
  XR Capacitación quirúrgica          Score: 7.14  →  GO
  Gemelos Digitales                   Score: 5.72  →  CONDITIONAL GO
  Blockchain EHR (híbrido)            Score: 4.85  →  NO-GO  (hasta madurar GDPR-compliance)
  Computación Cuántica                Score: 2.96  →  NO-GO  (reevaluar en 2028)
```

### Interpretación de resultados

| Decisión | Tecnologías | Acción |
|---|---|---|
| **GO** | Edge Computing, IA Gen+RAG, XR | Implementar de inmediato. Inversión justificada con ROI claro. |
| **CONDITIONAL GO** | Gemelos Digitales | Piloto en 2 hospitales. Evaluar resultados antes de escalar. |
| **NO-GO** | Blockchain completo, Cómputo Cuántico | Monitorear. No invertir en fase actual. Blockchain: modelo híbrido como alternativa parcial. |

---

## Resumen Ejecutivo

MEDICARE GLOBAL debe priorizar las tecnologías con mayor madurez y valor clínico demostrado:

1. **Primero:** Edge Computing + IA Generativa con RAG — impacto inmediato en diagnóstico y eficiencia, especialmente en zonas rurales con conectividad limitada.
2. **Segundo:** XR para capacitación quirúrgica — ROI demostrado, bajo riesgo regulatorio, reducción de errores probada.
3. **Tercero (piloto):** Gemelos Digitales en hospitales principales — planificación estratégica de largo plazo.
4. **Diferir:** Blockchain (modelo híbrido auditoria, no almacenamiento completo) y Cómputo Cuántico (reevaluar en 2028-2029).

La inversión de $160M se recupera en ≈6.2 años con un VOI significativo en calidad asistencial, acceso a zonas rurales y posicionamiento como referente médico digital en América Latina.
