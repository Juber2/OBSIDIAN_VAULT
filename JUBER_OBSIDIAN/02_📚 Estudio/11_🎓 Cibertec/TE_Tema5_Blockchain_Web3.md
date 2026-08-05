---
tags:
  - Cibertec
  - TecnologiasEmergentes
  - Tema5
Actualizado: 2026-06-23
Curso: "[[Tecnologias_Emergentes]]"
---

# Tema 5 — Blockchain/Web3 y Casos de Uso Empresariales

> Blockchain combina descentralización, inmutabilidad y criptografía para registrar, verificar y automatizar transacciones digitales sin intermediarios.

---

## 1. Fundamentos: DLT, Contratos Inteligentes y Tokenización

### DLT vs Blockchain

| Concepto | Definición |
|----------|------------|
| **DLT** (Distributed Ledger Technology) | Tecnología que distribuye y sincroniza un registro entre varios nodos, **sin depender de un servidor central**. Todos los participantes validan las transacciones mediante consenso. |
| **Blockchain** | Tipo específico de DLT que organiza la información en **bloques encadenados criptográficamente**, asegurando que los datos sean transparentes, verificables e inmutables. |

---

### 5 Características clave de DLT y Blockchain

| Característica | Descripción |
|----------------|-------------|
| **Descentralización** | Los datos se distribuyen entre diversos nodos, reduciendo dependencias y eliminando intermediarios. No hay un punto único de control o fallo. |
| **Transparencia** | Las transacciones pueden ser consultadas por los participantes autorizados, facilitando auditorías y trazabilidad. En blockchains públicas, cualquiera puede verificar. |
| **Inmutabilidad** | Una vez registrada, la información **no puede modificarse** sin obtener el consenso de la red. Cualquier alteración rompe la cadena criptográfica. |
| **Seguridad criptográfica** | Cada bloque incorpora firmas digitales y funciones hash que aseguran la integridad de los datos. Cambiar un bloque invalida todos los siguientes. |
| **Mecanismos de consenso** | La validación de transacciones se realiza mediante algoritmos como Proof of Work (PoW), Proof of Stake (PoS) u otros protocolos orientados a eficiencia energética y escalabilidad. |

---

### Estructura y Funcionamiento de la Cadena de Bloques

#### 3 Elementos esenciales

| Elemento | Descripción |
|----------|-------------|
| **Bloques** | Agrupan un conjunto de transacciones ya validadas. Contienen: un hash único identificador, el hash del bloque anterior (encadenamiento), una marca de tiempo y metadatos del proceso de validación. |
| **Nodos** | Participantes de la red que almacenan una copia del registro completo y ejecutan las funciones de verificación, consenso y propagación de nuevos bloques. Pueden ser nodos completos, ligeros u operadores especializados. |
| **Mecanismo de consenso** | Reglas y algoritmos mediante los cuales los nodos acuerdan qué transacciones son válidas y qué bloques se incorporan. Los más comunes: **PoW**, **PoS** y **DPoS** (Delegated Proof of Stake). |

#### Flujo general de operación (5 pasos)

1. Un usuario genera una transacción y la envía a la red.
2. Los nodos validan la transacción siguiendo las reglas del protocolo (firmas digitales, saldos, permisos).
3. Las transacciones válidas se agrupan en un nuevo bloque.
4. El bloque obtiene su hash y se enlaza criptográficamente al bloque anterior.
5. La actualización se replica y sincroniza en todos los nodos, asegurando que cada copia del registro sea idéntica.

> Este mecanismo descentralizado garantiza que la blockchain opere como un sistema **transparente, verificable e inmutable**, eliminando la necesidad de intermediarios.

---

### Contratos Inteligentes (Smart Contracts)

Programas informáticos almacenados en la blockchain que **ejecutan automáticamente acuerdos o reglas** cuando se cumplen determinadas condiciones. Funcionan como contratos autoejecutables donde las cláusulas están escritas en código y se validan sin intervención humana.

#### 4 Características principales

| Característica | Descripción |
|----------------|-------------|
| **Automatización** | Las acciones se ejecutan automáticamente al cumplirse las condiciones, reduciendo tiempos y errores humanos. |
| **Transparencia** | El código y las transacciones pueden ser auditados por todas las partes involucradas. |
| **Inmutabilidad** | Una vez desplegado, el contrato no puede modificarse sin consenso, garantizando integridad y confiabilidad. |
| **Seguridad** | El uso de criptografía y validación distribuida previene manipulaciones o ejecuciones indebidas. |

> **Ejemplo práctico:** Un contrato inteligente puede liberar un pago a un proveedor cuando un sensor IoT confirma la entrega de un producto. Todo el proceso ocurre sin validaciones manuales, reduciendo costos y agilizando la operación.

#### Plataformas principales para Smart Contracts

| Plataforma | Características |
|------------|----------------|
| **Ethereum** | La más usada. EVM (Ethereum Virtual Machine), lenguaje Solidity. Ecosistema DeFi y NFT muy desarrollado. |
| **Solana** | Alta velocidad (65,000 TPS), bajo costo por transacción. Ideal para aplicaciones de alto rendimiento. |
| **Hyperledger Fabric** | Diseñada para uso empresarial privado/permisionado. Control de acceso granular, sin criptomoneda nativa. |
| **Polkadot** | Especializada en interoperabilidad entre blockchains mediante su arquitectura de paracadenas. |
| **Cardano** | Enfoque académico y verificación formal. Usa PoS (Ouroboros). Alta seguridad matemáticamente demostrada. |

> Los contratos inteligentes habilitan ecosistemas digitales autónomos y confiables, transformando procesos como pagos, certificaciones, trazabilidad y gobernanza distribuida.

---

### Tokenización de Activos

Consiste en representar activos físicos o digitales mediante **unidades digitales llamadas tokens**, registradas en una blockchain. Pueden transferirse, fraccionarse o intercambiarse sin intermediarios, manteniendo un registro seguro, transparente y verificable de la propiedad.

#### 4 Tipos de tokens

| Tipo | Descripción | Ejemplos |
|------|-------------|---------|
| **Tokens fungibles (FTs)** | Activos digitales intercambiables entre sí con el mismo valor. | Criptomonedas (Bitcoin, ETH), puntos de fidelidad, créditos digitales. |
| **Tokens no fungibles (NFTs)** | Representan activos únicos e irrepetibles, cada uno con identidad propia. | Arte digital, certificados académicos, identidad digital, patentes, registros únicos. |
| **Security tokens** | Instrumentos financieros tokenizados (acciones, bonos, fondos). Están regulados y otorgan derechos financieros y de gobernanza. | Acciones tokenizadas, bonos digitales, fondos de inversión on-chain. |
| **Utility tokens** | Permiten acceder a servicios, productos o funciones dentro de un ecosistema digital. No representan propiedad, sino derechos de uso. | Tokens de acceso a plataformas, gaming, servicios cloud descentralizados. |

#### Beneficios de la tokenización

| Beneficio | Descripción |
|-----------|-------------|
| **Liquidez** | Permite fraccionar activos ilíquidos (inmuebles, obras de arte, infraestructura) y negociarlos en mercados digitales. |
| **Accesibilidad** | Reduce barreras de entrada: cualquier persona puede invertir en fracciones de activos antes reservados a grandes capitales. |
| **Transparencia** | El historial de propiedad y transferencias queda registrado de forma inmutable y verificable en la blockchain. |
| **Automatización** | Los contratos inteligentes gestionan derechos, pagos de dividendos, transferencias y cumplimiento de forma autónoma. |
| **Reducción de intermediarios** | Elimina brokers, custodios y procesos burocráticos, reduciendo costos y tiempos de liquidación. |

---

## 2. Identidad Digital, Trazabilidad y Registros Inmutables

### Identidad Descentralizada (DID — Decentralized Identity)

En los sistemas tradicionales, la gestión centralizada de identidad genera dependencia, riesgos de privacidad y mayor exposición a ciberataques. Con **Web3**, la identidad adopta un modelo **descentralizado** donde el usuario controla sus propias credenciales.

#### 5 Principios de la identidad descentralizada

| Principio | Descripción |
|-----------|-------------|
| **Propiedad individual** | El usuario es el único dueño de su identidad y controla sus credenciales. Nadie puede revocarlas sin su consentimiento. |
| **Verificabilidad descentralizada** | Terceros pueden validar credenciales sin intermediarios, usando mecanismos criptográficos. |
| **Interoperabilidad** | Una misma identidad puede utilizarse en múltiples servicios o plataformas sin necesidad de cuentas separadas. |
| **Privacidad selectiva** | El usuario decide qué datos comparte. Se aplican modelos como **zero-knowledge proofs** (ZKP) para probar información sin revelarla. |
| **Seguridad criptográfica** | Las credenciales se protegen mediante claves criptográficas y registros inmutables en blockchain. |

#### 3 Componentes del modelo DID

| Componente | Descripción |
|------------|-------------|
| **DID** (Decentralized Identifier) | Identificador único, persistente y verificable que representa a una persona, entidad u objeto dentro de una red blockchain o DLT. No depende de un proveedor central y puede resolverse criptográficamente. |
| **VC** (Verifiable Credential) | Conjunto de atributos o afirmaciones emitidas por una entidad confiable (diploma, certificado médico, licencia). Incluyen mecanismos criptográficos que permiten verificar autenticidad sin revelar información innecesaria. |
| **Wallet de identidad digital** | Aplicación o entorno seguro donde el usuario almacena, administra y presenta sus credenciales verificables. Funciona como un "billetero" digital autocustodiado que garantiza el control exclusivo del usuario sobre sus datos. |

> Estos componentes permiten crear ecosistemas de autenticación y verificación **sin intermediarios**, donde las identidades son portables, seguras y resistentes a la censura o manipulación.

---

### Trazabilidad en Blockchain

Capacidad de **rastrear el origen, la evolución y el estado actual** de un activo, dato o transacción, garantizando su autenticidad e integridad a lo largo del tiempo. Cada transacción registrada forma parte de un historial **permanente e inmutable**, accesible para auditorías o verificaciones sin intermediarios.

#### Beneficios de la trazabilidad blockchain

| Beneficio | Descripción |
|-----------|-------------|
| **Visibilidad completa** | Permite seguir el recorrido de un producto, documento o activo desde su origen hasta el destino final. |
| **Reducción de fraude** | La inmutabilidad del registro hace prácticamente imposible alterar o falsificar el historial sin ser detectado. |
| **Eficiencia en auditorías** | Las auditorías se realizan en tiempo real sobre datos verificables, reduciendo tiempos y costos de revisión. |
| **Cumplimiento normativo** | Facilita la demostración de conformidad con regulaciones sectoriales (trazabilidad alimentaria, farmacéutica, etc.). |
| **Confianza entre partes** | Elimina la necesidad de confianza entre contrapartes; el registro neutral de blockchain actúa como árbitro. |

---

### Registros Inmutables y Verificación de Integridad

Una de las propiedades más distintivas de blockchain es mantener registros que **no pueden modificarse ni eliminarse** una vez validados. Cada bloque incluye un hash criptográfico (huella digital única) que enlaza con el bloque anterior. Cualquier alteración **rompe la cadena**, haciendo evidente la manipulación.

#### Beneficios de los registros inmutables

| Beneficio | Descripción |
|-----------|-------------|
| **Integridad garantizada** | Los datos registrados son exactamente los que fueron validados; no pueden ser alterados retroactivamente. |
| **No repudio** | Ninguna parte puede negar haber realizado una transacción, ya que queda registrada permanentemente. |
| **Auditoría permanente** | Cualquier participante autorizado puede verificar el historial completo en cualquier momento. |
| **Confianza sistémica** | La confianza no depende de una autoridad central sino de la matemática criptográfica y el consenso distribuido. |

> La blockchain se posiciona como tecnología idónea para sistemas de **certificación, auditoría, seguimiento y control documental** en múltiples industrias.

---

## 3. Escalabilidad, Interoperabilidad y Consumo Energético

### El Trilema de la Blockchain

La blockchain enfrenta un dilema fundamental conocido como el **trilema**: es difícil optimizar simultáneamente los tres pilares:

```
Seguridad ◄──────────────────────────────────────────► Descentralización
                          ▲
                          │
                   Escalabilidad
```

Mejorar uno de los tres pilares suele implicar sacrificar alguno de los otros dos.

---

### Escalabilidad

Capacidad de procesar más transacciones por segundo (TPS) sin afectar la seguridad ni la descentralización.

**Limitaciones actuales:** bloques pequeños o lentos, validación en todos los nodos (añade latencia), tiempo de sincronización en toda la red.

#### 4 Estrategias de escalabilidad

| Estrategia | Descripción |
|------------|-------------|
| **Cadenas laterales (Sidechains)** | Redes paralelas que procesan transacciones fuera de la cadena principal, manteniendo seguridad mediante anclajes periódicos. |
| **Soluciones de segunda capa (Layer 2)** | Mecanismos como **Lightning Network**, **Optimistic Rollups** o **ZK-Rollups** que ejecutan transacciones fuera de la cadena principal y solo registran pruebas o resultados agregados. |
| **Fragmentación (Sharding)** | Divide la red en grupos más pequeños (shards) capaces de procesar transacciones en paralelo, aumentando el throughput total. |
| **Algoritmos de consenso más eficientes** | Mecanismos como Proof of Stake (PoS) y sus variantes reducen la carga computacional y aceleran la validación. |

---

### Interoperabilidad

Capacidad de diferentes redes blockchain o sistemas distribuidos para **intercambiar información y operar de forma conjunta**. Sin esta capacidad, cada cadena actúa como un ecosistema aislado, limitando el potencial de las aplicaciones Web3.

#### 3 Niveles de interoperabilidad

| Nivel | Descripción |
|-------|-------------|
| **Interoperabilidad de datos** | Permite que distintas blockchains intercambien información de manera segura y verificable, manteniendo autenticidad y consistencia entre redes. |
| **Interoperabilidad de activos** | Facilita la transferencia de tokens, criptomonedas o activos digitales entre cadenas mediante mecanismos seguros como **bridges** o sidechains. |
| **Interoperabilidad de contratos inteligentes** | Permite que aplicaciones descentralizadas (dApps) ejecuten funciones en múltiples redes, ampliando su alcance y flexibilidad. |

> La interoperabilidad es un habilitador fundamental para: finanzas descentralizadas (DeFi), trazabilidad, metaverso, identidad digital y tokenización. Sin ella, la adopción empresarial queda fragmentada y limitada.

---

### Consumo Energético

Los sistemas basados en **Proof of Work (PoW)** requieren gran potencia computacional para validar transacciones, generando preocupaciones ambientales y regulatorias.

#### Impactos del consumo energético

| Impacto | Descripción |
|---------|-------------|
| **Huella de carbono elevada** | Las operaciones mineras de redes PoW consumen tanta energía como países enteros (Bitcoin ~ consumo de Argentina). |
| **Costos operativos altos** | El costo energético de la minería presiona los márgenes de los validadores y puede limitar la economía del protocolo. |
| **Presión regulatoria** | Gobiernos y organismos internacionales imponen restricciones o incentivos negativos a blockchains con alto consumo. |
| **Limitación de adopción** | Las empresas con objetivos de sostenibilidad (ESG) evitan asociarse con tecnologías de alto impacto ambiental. |

#### 5 Alternativas para reducir el consumo energético

| Alternativa | Descripción |
|-------------|-------------|
| **Proof of Stake (PoS)** | Reduce el uso intensivo de cómputo al elegir validadores según los tokens en stake. Disminuye el consumo energético **hasta un 99%** (Ethereum redujo su consumo 99.95% al migrar a PoS). |
| **Proof of Authority (PoA)** | Usa validadores confiables y verificados. Ideal para blockchains privadas o permisionadas con prioridad en eficiencia. |
| **Proof of Space / Proof of Time** | Reemplaza el procesamiento intensivo por almacenamiento o sincronización temporal para reducir la demanda computacional. |
| **Energía renovable** | Centros de datos y operaciones mineras integran fuentes eólicas, solares o hidroeléctricas para mitigar el impacto ambiental. |
| **Optimización del protocolo** | Reducción del tamaño de bloque, validación parcial y soluciones de Layer 2 para descomprimir la cadena principal. |

---

## 4. Riesgos, Cumplimiento y Auditoría en Entornos Web3

El ecosistema Web3 propone un modelo descentralizado donde las decisiones recaen en redes distribuidas. Esta estructura trae nuevos riesgos: procesos autónomos, **código inmutable** y responsabilidades compartidas entre múltiples actores.

### 3 Dimensiones de riesgo en Web3

| Tipo | Descripción |
|------|-------------|
| **Riesgos técnicos** | Fallas en el código, errores en contratos inteligentes, debilidades criptográficas o vulnerabilidades en la arquitectura. Pueden comprometer la integridad del sistema y permitir ataques o pérdidas de activos. |
| **Riesgos operativos** | Errores humanos, mala gestión de claves privadas, falta de controles adecuados o fallas en los procesos de administración. Afectan la continuidad y seguridad de las operaciones. |
| **Riesgos legales y éticos** | Incumplimiento normativo, riesgos de lavado de activos, problemas de privacidad, regulación deficiente de activos digitales o usos indebidos de funciones autónomas de la tecnología. |

---

### 5 Marcos Regulatorios principales en Web3

| Marco | Descripción |
|-------|-------------|
| **AI Act** (UE, 2024) | Regula el uso seguro, ético y supervisado de sistemas de inteligencia artificial, incluidos aquellos que interactúan con blockchain. |
| **MiCA** (Markets in Crypto-Assets, UE) | Define reglas para emisores de tokens, exchanges y proveedores de servicios relacionados con criptoactivos. Primer marco regulatorio integral para criptoactivos. |
| **GDPR** (UE) | Protege los datos personales y plantea retos importantes en blockchain por la **inmutabilidad** de la información registrada (derecho al olvido vs. registros permanentes). |
| **FATF** (Financial Action Task Force) | Establece lineamientos contra el lavado de activos y financiamiento ilícito mediante criptoactivos. Incluye la **"travel rule"**: intercambio de datos del remitente y receptor en transacciones cripto. |
| **ISO/TC 307** | Conjunto de normas internacionales para gobernanza, seguridad, auditoría e interoperabilidad de sistemas blockchain. |

---

### 4 Tipos de Auditoría en Blockchain y Web3

La auditoría en Web3 consiste en la **revisión, validación y verificación continua** de los componentes técnicos, operativos y transaccionales de una red o aplicación descentralizada.

| Tipo | Descripción |
|------|-------------|
| **Auditoría de contratos inteligentes** | Revisión del código antes y después de su despliegue para identificar vulnerabilidades, errores lógicos o posibles comportamientos maliciosos. Incluye análisis estático, dinámico y verificación formal. |
| **Auditoría de seguridad de red** | Evalúa la robustez de la infraestructura: nodos, mecanismos de consenso, oráculos, bridges, gestión de claves y canales de comunicación. Objetivo: prevenir ataques como reentrancy, manipulación de oráculos o intentos de toma de control. |
| **Auditoría de cumplimiento** | Verifica que las operaciones y políticas del proyecto cumplan con normativas locales e internacionales (GDPR, MiCA, KYC/AML). Especialmente relevante para tokens, exchanges y plataformas DeFi. |
| **Auditoría automatizada (on-chain)** | Uso de herramientas que monitorean en tiempo real las transacciones, estados de contratos inteligentes o eventos relevantes. Permite detectar anomalías, accesos inusuales o patrones sospechosos sin intervención humana. |

> Las auditorías son un **pilar esencial** para la adopción empresarial de Web3: aseguran la integridad del ecosistema, garantizan el cumplimiento regulatorio y fortalecen la confianza entre participantes.

---

## Ideas clave del Tema

1. La evolución de blockchain exige enfrentar desafíos técnicos, operativos y ambientales para garantizar redes más **eficientes, seguras y sostenibles** en escenarios de uso masivo.
2. La **escalabilidad, la interoperabilidad y el consumo energético** son factores críticos que determinan la viabilidad de los ecosistemas Web3 y su integración con aplicaciones empresariales.
3. Las soluciones emergentes —**PoS, Layer 2, sidechains e infraestructura verde**— permiten avanzar hacia un Web3 más responsable, reduciendo costos, riesgos y el impacto ambiental sin sacrificar seguridad ni descentralización.
