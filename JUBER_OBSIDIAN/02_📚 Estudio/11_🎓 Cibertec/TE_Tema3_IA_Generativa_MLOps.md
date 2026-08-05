---
tags:
  - Cibertec
  - TecnologiasEmergentes
  - Tema3
Actualizado: 2026-06-23
Curso: "[[Tecnologias_Emergentes]]"
---

# Tema 3 — IA Generativa y MLOps

> La adopción de IA requiere combinar innovación, calidad operativa y responsabilidad en el desarrollo de tecnologías avanzadas.

---

## 1. LLMs, Prompt Engineering y RAG

### IA Generativa

La inteligencia artificial generativa permite **crear contenido nuevo** —texto, imágenes, audio o código— a partir de patrones aprendidos en grandes volúmenes de datos. A diferencia de la IA tradicional (que clasifica o predice), los modelos generativos **producen información coherente y contextual** para diversas tareas.

En el procesamiento del lenguaje natural esta capacidad se expresa en los **Large Language Models (LLMs)**.

---

### Modelos de Lenguaje a Gran Escala (LLMs)

Sistemas de IA entrenados con enormes volúmenes de texto para **predecir la siguiente palabra en una secuencia**. Se basan en arquitecturas **Transformer**, que permiten identificar patrones lingüísticos y comprender contextos amplios con alta precisión.

> Ejemplos: ChatGPT, Claude, Gemini.

#### Características principales de los LLMs

| Característica | Descripción |
|----------------|-------------|
| **Entrenamiento masivo** | Entrenados con cientos de miles de millones de tokens de texto provenientes de libros, webs, código y documentos. |
| **Arquitectura Transformer** | Mecanismo de atención (attention) que permite al modelo considerar el contexto completo de la secuencia, no solo las palabras adyacentes. |
| **Comprensión contextual amplia** | Capaces de mantener coherencia en conversaciones largas y comprender instrucciones complejas con múltiples condiciones. |
| **Generalización multi-tarea** | Un mismo modelo puede redactar, resumir, traducir, programar, analizar o responder preguntas sin entrenamiento específico para cada tarea. |
| **Afinación y adaptación** | Pueden especializarse mediante fine-tuning (entrenamiento adicional con datos específicos del dominio) o RLHF (Reinforcement Learning from Human Feedback). |
| **Generación probabilística** | Producen texto de forma estocástica: el resultado puede variar entre ejecuciones. El parámetro *temperatura* controla la creatividad vs. consistencia de las respuestas. |

#### Aplicaciones principales de los LLMs

| Aplicación | Descripción |
|------------|-------------|
| **Generación y redacción de texto** | Creación de artículos, correos, informes, contenido de marketing o documentación técnica. |
| **Resumen y síntesis** | Condensar documentos extensos, reportes o conversaciones en resúmenes precisos. |
| **Traducción automática** | Traducción de alto contexto entre idiomas con comprensión semántica. |
| **Generación de código** | Asistencia en programación: autocompletado, generación, revisión, explicación y debugging de código. |
| **Análisis y clasificación** | Análisis de sentimiento, categorización de documentos, extracción de entidades y datos. |
| **Chatbots y asistentes virtuales** | Sistemas conversacionales con memoria de contexto para soporte, onboarding o atención al cliente. |
| **Respuesta a preguntas (Q&A)** | Consultas sobre bases de conocimiento internas, manuales, normativas o documentación técnica. |

---

### Prompt Engineering: Diseño Efectivo de Entradas

Consiste en **diseñar y optimizar las instrucciones** que se entregan a un modelo de lenguaje para obtener respuestas precisas, útiles y controladas.

> Un **prompt** es el texto de entrada que guía el comportamiento del modelo. Su calidad determina la pertinencia, claridad y coherencia del resultado generado.

**Objetivo:** reducir la ambigüedad, mejorar la relevancia de las respuestas y garantizar que la salida del modelo se alinee con los requerimientos de la tarea o del proceso organizacional.

#### 6 Principios del Prompt Engineering

| # | Principio | Descripción |
|---|-----------|-------------|
| 1 | **Claridad y especificidad** | Formular instrucciones directas con objetivos concretos. Evitar formulaciones vagas o ambiguas. |
| 2 | **Contexto suficiente** | Incluir información necesaria: rol del modelo, propósito de la tarea, público objetivo, restricciones o limitaciones. |
| 3 | **Estructuración de tareas complejas** | Dividir actividades amplias en pasos secuenciales (decomposición), facilitando que el modelo siga un proceso lógico. |
| 4 | **Control del estilo y formato** | Solicitar explícitamente el tono, extensión, estructura o tipo de salida deseada (lista, resumen, tabla, paso a paso, JSON, etc.). |
| 5 | **Ejemplificación (few-shot prompting)** | Incluir ejemplos de entradas y salidas esperadas para que el modelo replique el patrón deseado. |
| 6 | **Iteración y refinamiento** | Ajustar los prompts a partir de los resultados obtenidos, afinando progresivamente el rendimiento en función de los objetivos. |

> El dominio del prompt engineering permite mejorar la eficacia de los modelos **sin necesidad de modificar su arquitectura ni realizar fine-tuning**. Es una competencia clave para implementar IA generativa de forma responsable, reproducible y alineada con el negocio.

---

### Retrieval-Augmented Generation (RAG)

**Problema que resuelve:** el conocimiento de los LLMs se restringe a los datos con los que fueron entrenados. Esto puede generar respuestas incompletas o desactualizadas, especialmente con información reciente o específica de un dominio.

**Solución RAG:** combina dos capacidades:
1. **Recuperación de información relevante** desde fuentes externas (bases de datos, documentos, repositorios)
2. **Generación de texto** mediante un modelo de lenguaje

Esta integración permite que el sistema **amplíe su base de conocimiento en tiempo real**, mejorando la precisión, coherencia y verificabilidad de las respuestas.

#### Proceso de funcionamiento del RAG

| Paso | Acción |
|------|--------|
| 1 | El usuario envía una **consulta** al sistema. |
| 2 | El sistema convierte la consulta en un **vector semántico** y lo compara contra una base de conocimiento indexada (vector store). |
| 3 | Se recuperan los **fragmentos más relevantes** de documentos, bases de datos o repositorios internos. |
| 4 | Los fragmentos recuperados se adjuntan al **prompt del LLM como contexto adicional**. |
| 5 | El LLM **genera una respuesta** combinando el contexto recuperado con su conocimiento base, citando fuentes cuando es posible. |

#### Casos de uso principales

- Sistemas de **soporte técnico** con acceso a manuales actualizados
- **Automatización documental** (contratos, reportes, normativas)
- **Motores de búsqueda inteligentes** sobre bases de conocimiento internas
- **Asistentes corporativos** basados en información confiable y verificable

> RAG se ha consolidado como un **componente central** de las arquitecturas modernas de IA empresarial.

---

## 2. Ciclo de Vida y Trazabilidad de Datos, Modelos y Versiones

El desarrollo de soluciones de IA es un **proceso continuo** que va más allá del entrenamiento de modelos. Su ciclo de vida define cómo se gestionan los datos, modelos, pruebas, versiones y operación, asegurando soluciones **transparentes, trazables, auditables y sostenibles**.

### Las 5 Fases del Ciclo de Vida de un Proyecto de IA

| # | Fase | Descripción |
|---|------|-------------|
| 1 | **Recolección y preparación de datos** | Identificación de fuentes relevantes, limpieza, normalización y documentación de procedencia y licencias. La calidad del modelo **depende directamente de esta fase**. |
| 2 | **Exploración y análisis de datos** | Comprensión del dominio, análisis estadístico, detección de sesgos, valores atípicos y patrones. |
| 3 | **Entrenamiento y validación del modelo** | Selección de algoritmos y arquitectura, entrenamiento supervisado o generativo, validación cruzada y ajuste de hiperparámetros. |
| 4 | **Despliegue e integración** | Implementación en entornos controlados o productivos, integración con flujos de negocio y monitoreo de rendimiento y costos. |
| 5 | **Mantenimiento y mejora continua** | Actualización periódica de datos, retrenamiento, gestión de versiones y reemplazo del modelo cuando sea necesario. |

---

### Trazabilidad

Consiste en **registrar y seguir cada etapa del ciclo de vida** de los datos y modelos de un sistema de IA. Su objetivo es asegurar transparencia, reproducibilidad y responsabilidad técnica, especialmente cuando la IA influye en decisiones operativas o estratégicas.

#### 5 Elementos clave de la trazabilidad

| Elemento | Descripción |
|----------|-------------|
| **Origen y calidad de los datos** | Registro de fuentes utilizadas (internas y externas), tipos de licenciamiento, condiciones de uso, criterios de selección y verificaciones de integridad. |
| **Transformaciones realizadas** | Documentación detallada de los procesos de limpieza, normalización, enriquecimiento, anonimización y particionamiento, incluyendo versiones de scripts o pipelines. |
| **Configuración del modelo** | Parámetros de entrenamiento, arquitectura utilizada, algoritmos seleccionados, librerías, versiones de frameworks y características del entorno computacional. |
| **Evidencias de desempeño** | Resultados de validación, métricas (precisión, recall, F1, AUC, entre otros), análisis de sesgo y pruebas de robustez, manteniendo historial de todas las iteraciones. |
| **Historial de decisiones técnicas** | Razones detrás de cada cambio o ajuste significativo, incluyendo descartes de modelos previos, modificaciones de criterios y decisiones de gobernanza técnica. |

---

### Control de Versiones en IA

El versionado consiste en **registrar sistemáticamente los cambios en datos, modelos y código** para rastrear su evolución y recuperar versiones previas cuando sea necesario. Evita inconsistencias, fortalece la trazabilidad y facilita auditorías y cumplimiento normativo.

#### Buenas prácticas de versionado

| Práctica | Descripción |
|----------|-------------|
| **Versionado de datos** | Mantener copias identificables de los distintos conjuntos de datos usados en entrenamiento, validación y prueba. Cada versión debe incluir metadatos clave: fuente, fecha, licencia, transformaciones aplicadas y justificación de cambios. |
| **Versionado de modelos** | Registrar arquitecturas, configuraciones de entrenamiento, hiperparámetros, pesos y métricas de desempeño. Permite comparar experimentos, explicar diferencias entre versiones y garantizar que el modelo desplegado sea verificable. |
| **Herramientas de control de versiones** | Usar plataformas que integren datos, código y modelos: **Git**, **DVC** (Data Version Control), **MLflow**, **Weights & Biases** o **Kubeflow**. Facilitan la colaboración y la estandarización operativa. |
| **Versiones estables y etiquetado** | Identificar explícitamente las versiones aprobadas para producción mediante tags o ramas específicas. Permite separar los experimentos en desarrollo de las implementaciones confiables que cumplen con requisitos de calidad, seguridad y rendimiento. |

> El control de versiones es un **pilar fundamental del MLOps moderno**: garantiza la consistencia entre entornos, respalda la trazabilidad regulatoria y permite evolucionar modelos y datos de manera segura y eficiente.

---

## 3. MLOps: Orquestación, CI/CD, Monitoreo y Evaluación

### ¿Qué es MLOps?

**Machine Learning Operations (MLOps)** es una extensión especializada de las prácticas DevOps, adaptada a las particularidades del aprendizaje automático. Su propósito es **estandarizar y automatizar el ciclo de vida completo de los modelos**, asegurando:

- **Reproducibilidad** de experimentos y resultados
- **Escalabilidad** para atender demandas reales de operación
- **Monitoreo continuo** para detectar degradación, sesgos o fallos
- **Gobernanza y trazabilidad** de datos, modelos y decisiones

---

### 5 Componentes principales de MLOps

| Componente | Descripción |
|------------|-------------|
| **Automatización** | Reduce tareas manuales mediante pipelines que cubren desde la preparación de datos hasta el entrenamiento y validación del modelo. Permite reproducir experimentos, minimizar errores y acelerar entregas. |
| **Orquestación** | Coordina tareas interdependientes —preprocesamiento, entrenamiento, evaluación, empaquetado y despliegue— asegurando que cada paso se ejecute en el orden correcto y con los recursos adecuados. Herramientas: Airflow, Kubeflow, Prefect. |
| **Integración y entrega continua (CI/CD)** | Adapta prácticas DevOps al ecosistema de IA para habilitar actualizaciones frecuentes y seguras. Incluye pruebas automatizadas, validación de desempeño, control de versiones y despliegues progresivos. |
| **Monitoreo y evaluación** | Supervisa modelos en producción, detectando desviaciones, data drift, deterioro del rendimiento o problemas de disponibilidad. Permite activar alertas, reentrenar modelos o aplicar correcciones. |
| **Gobernanza y cumplimiento** | Políticas para garantizar trazabilidad, auditoría, explicabilidad, control de versiones y cumplimiento normativo. Asegura que los modelos operen con estándares éticos, regulatorios y de seguridad. |

---

### Orquestación de Flujos de Trabajo

La orquestación permite **coordinar, automatizar y supervisar** todas las tareas involucradas en el desarrollo, entrenamiento y despliegue de modelos de IA. Garantiza que cada etapa del pipeline ocurra en el orden correcto, con los recursos necesarios y bajo condiciones reproducibles.

#### Etapas típicas de un flujo orquestado

```
Ingesta de datos → Preprocesamiento → Entrenamiento → Evaluación → Empaquetado → Despliegue → Monitoreo
```

#### Herramientas comunes de orquestación

| Herramienta | Ventajas principales |
|-------------|---------------------|
| **Apache Airflow** | DAGs (grafos dirigidos acíclicos) para orquestar workflows de datos. Amplia comunidad, flexible e integrable con múltiples plataformas. |
| **Kubeflow** | Orquestación nativa en Kubernetes para ML. Ideal para entornos cloud-native y cargas de trabajo distribuidas. |
| **Prefect** | Workflows modernos con mejor manejo de errores, observabilidad y recuperación automática. Más sencillo que Airflow. |
| **MLflow** | Tracking de experimentos, registro (Model Registry) y despliegue de modelos. Integrable con múltiples frameworks (PyTorch, TensorFlow, scikit-learn). |
| **Metaflow** (Netflix) | Diseñado para flujos de ciencia de datos. Manejo transparente de infraestructura cloud y versiones de datos. |

---

### Monitoreo y Evaluación de Modelos en Producción

Un modelo desplegado requiere **monitoreo constante** para asegurar su rendimiento y confiabilidad. Permite detectar **data drift** (cambios en los datos) o **concept drift** (cambios en el entorno), que pueden reducir la precisión del modelo.

#### 5 Aspectos a monitorear

| Aspecto | Qué se mide |
|---------|-------------|
| **Desempeño del modelo** | Métricas como accuracy, precision, recall, F1-score, MAE o AUC según la tarea. Comparación contra la línea base del entrenamiento. |
| **Disponibilidad operativa** | Tiempos de respuesta, latencia, errores de inferencia, caídas del servicio y cumplimiento de SLAs. |
| **Integridad y uso de recursos** | Consumo de CPU, memoria, throughput y costos asociados en la nube o infraestructura propia. |
| **Deriva de datos y concepto** | Comparación continua entre distribuciones actuales y las del entrenamiento. Identificación de cambios en patrones o correlaciones. |
| **Equidad y sesgos emergentes** | Verificación de que el rendimiento es **consistente entre distintos grupos de usuarios**, detectando discriminación algorítmica emergente. |

> El monitoreo continuo, complementado con procesos de **reevaluación y retraining**, permite mantener modelos actualizados, confiables y alineados con las condiciones reales del negocio.

---

## 4. Riesgos en IA: Sesgos, Seguridad y Salvaguardas Técnicas

> Los riesgos en IA pueden aparecer en **cualquier etapa del ciclo de vida** — desde la recolección de datos hasta la operación en producción — y afectan la confiabilidad, la seguridad, la ética y el cumplimiento normativo.

---

### Sesgos Algorítmicos

Se producen cuando los modelos de IA **reflejan o amplifican prejuicios** presentes en los datos o en su diseño. Afectan la equidad, precisión y legitimidad de las decisiones automatizadas, especialmente en entornos críticos como salud, justicia, empleo o servicios públicos.

#### 4 Fuentes de sesgo

| Tipo | Descripción |
|------|-------------|
| **Sesgo en los datos** | Ocurre cuando los datos de entrenamiento son incompletos, desbalanceados o no representan adecuadamente a la población real. *Ejemplo: dataset de reconocimiento facial con mayoría de rostros de una sola etnia.* |
| **Sesgo en el diseño** | Surge por seleccionar variables inadecuadas, arquitecturas incorrectas o supuestos erróneos durante el modelado. |
| **Sesgo en la evaluación** | Provocado por interpretar mal los resultados o usar métricas incorrectas al validar el modelo (p. ej., usar solo accuracy en datasets desbalanceados). |
| **Sesgo de interacción** | Aparece cuando el sistema se comporta de forma desigual con distintos grupos de usuarios debido a lenguaje, estilo o contexto cultural. |

#### Estrategias de mitigación de sesgos

| Estrategia | Descripción |
|------------|-------------|
| **Auditoría y diversificación de datos** | Revisar activamente la representatividad del dataset, enriquecerlo con datos de grupos subrepresentados y documentar sus limitaciones. |
| **Métricas de equidad (fairness metrics)** | Evaluar el modelo no solo por precisión global sino por su desempeño diferenciado en distintos grupos (equalized odds, demographic parity). |
| **Técnicas de debiasing** | Aplicar algoritmos de rebalanceo de datos (oversampling, undersampling), regularización o post-procesamiento de predicciones para reducir discriminación. |
| **Supervisión humana en decisiones críticas** | Implementar revisión humana obligatoria en decisiones de alto impacto (crédito, salud, justicia) antes de actuar sobre las predicciones del modelo. |
| **Pruebas de equidad por grupos** | Validar sistemáticamente el modelo en subpoblaciones específicas antes del despliegue. |
| **Monitoreo continuo en producción** | Detectar sesgos emergentes que aparecen cuando el modelo interactúa con usuarios reales en condiciones cambiantes. |

---

### Seguridad en IA

Busca **proteger modelos, datos y sistemas** frente a ataques o manipulaciones que puedan comprometer su funcionamiento. A diferencia de otros entornos digitales, los modelos de IA poseen **vulnerabilidades propias** ya que aprenden de los datos y pueden ser influenciados o alterados a través de ellos.

#### 5 Tipos de ataques específicos a sistemas de IA

| Ataque | Descripción |
|--------|-------------|
| **Ataques adversarios** | Inputs manipulados de forma imperceptible para humanos que engañan al modelo y lo llevan a producir predicciones incorrectas. *Ejemplo: imagen de un stop sign modificada para que el modelo la clasifique como algo distinto.* |
| **Data poisoning** | Contaminación intencional de los datos de entrenamiento para introducir comportamientos maliciosos, backdoors o degradar el rendimiento del modelo. |
| **Exfiltración de información** | Extracción de datos confidenciales del modelo a través de consultas diseñadas para revelar información del dataset de entrenamiento o de los pesos del modelo. |
| **Robo o replicación del modelo** | Copiar el comportamiento de un modelo propietario enviando múltiples consultas y entrenando un modelo sustituto con las respuestas obtenidas. |
| **Uso indebido de IA generativa** | Deepfakes, desinformación masiva, phishing personalizado, ingeniería social automatizada o generación de malware mediante LLMs. |

#### Estrategias de protección (salvaguardas técnicas)

| Salvaguarda | Descripción |
|-------------|-------------|
| **Validación y saneamiento de datos de entrada** | Implementación de controles automáticos que detectan anomalías, patrones maliciosos o datos fuera de distribución antes de que ingresen al modelo. |
| **Robustez adversaria** | Uso de técnicas como adversarial training o defensas basadas en detección de ataques para aumentar la resistencia del modelo frente a manipulaciones. |
| **Seguridad en el ciclo de vida del modelo** | Cifrado, gestión segura de llaves, control de acceso y políticas de permisos mínimos para proteger artefactos y componentes del modelo. |
| **Monitoreo continuo** | Detección temprana de comportamientos atípicos, variaciones inesperadas en las respuestas o uso inusual del sistema para activar alertas o respuestas automatizadas. |
| **Protección contra extracción de modelos** | Limitación de tasas de consulta (rate limiting), análisis del patrón de uso y técnicas como **watermarking** para prevenir la copia o replicación del modelo. |

> La seguridad debe integrarse desde la fase de diseño (**security by design**) hasta la operación continua del sistema, garantizando que los modelos permanezcan fiables, resistentes y protegidos frente a amenazas emergentes.

---

## Ideas clave del Tema

1. La **IA generativa y los LLMs** ofrecen capacidades avanzadas para crear, sintetizar y transformar información, pero requieren un diseño adecuado de prompts, datos de calidad y técnicas como RAG para garantizar resultados precisos, actualizados y alineados a los objetivos del usuario.
2. El **ciclo de vida de los modelos** y la aplicación de prácticas de **MLOps** —versionado, automatización, orquestación, monitoreo y gobernanza— son esenciales para desarrollar sistemas de IA confiables, escalables y sostenibles en entornos reales.
3. La adopción de IA genera **riesgos técnicos, éticos y de seguridad** que deben anticiparse mediante trazabilidad, controles robustos, mitigación de sesgos y protección frente a ataques. Implementar salvaguardas apropiadas es clave para asegurar un uso responsable y seguro.
