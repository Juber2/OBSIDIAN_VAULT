---
tags:
  - Estudio
  - Carrera
  - Roadmap
Creado: 2026-08-31
Relacionado:
  - PLAN DE CAPACITACION
  - 08 - Norte de Carrera - Camino a Solution Architect
---

# Perfil objetivo — puestos con peso

## El cambio de criterio (31/08/2026)

Hasta ahora el plan se calibraba con [[16 - Radar acumulativo de vacantes]]: **frecuencia** entre 15 vacantes. Eso optimiza para *cubrir muchas*, y arrastra requisitos de puestos que no querés (VB6, FoxPro, soporte, Android).

Nuevo criterio: **estar a la altura de los puestos con peso**, aunque sean menos. Cambia qué sube y qué baja.

## Los 4 puestos de referencia

Los que marcaste como *"pagan bien"*, *"está chévere"* o *"algún día quisiera"*:

| Puesto | Nivel | Señal |
|---|---|---|
| **Cuidafarma** — Backend Node + AWS | Senior dev | S/ 6,500-7,500, el mejor sueldo confirmado |
| **Synopsis** — Senior FullStack Java/Angular/Python + IA | Senior dev | Fábrica de software con agentes de IA |
| **TCS** — Software Architect Full Stack | Arquitecto | Cloud Native, DDD, EDA, hexagonal |
| **Stefanini** — Solution Architect | Arquitecto | Diseño end-to-end, AWS+Azure, DevSecOps |

## Qué tienen en común — la intersección real

Crucé los cuatro avisos requisito por requisito:

| Capacidad | Cuidafarma | Synopsis | TCS Arch | Stefanini Arch | |
|---|:-:|:-:|:-:|:-:|---|
| **APIs REST + OpenAPI/Swagger** | ✅ | ✅ | ✅ | ✅ | **4/4** |
| **Clean Architecture / SOLID / DDD** | ✅ | ✅ | ✅ | ✅ | **4/4** |
| **CI/CD** | ✅ | ✅ | ✅ | ✅ | **4/4** |
| **Git y trabajo colaborativo** | ✅ | ✅ | ✅ | ✅ | **4/4** |
| Microservicios | — | ✅ | ✅ | ✅ | 3/4 |
| Cloud práctico | ✅ AWS | — | ✅ | ✅ AWS+Azure | 3/4 |
| Seguridad (OAuth2/JWT/OWASP) | ✅ | — | ✅ | ✅ DevSecOps | 3/4 |
| Observabilidad y diagnóstico | ✅ | — | ✅ | ✅ | 3/4 |
| Testing (unit, integración, TDD) | ✅ | ✅ | ✅ | — | 3/4 |
| Event-driven, colas, idempotencia | ✅ | — | ✅ | ✅ | 3/4 |
| Java / Spring | — | ✅ | ✅ | — | 2/4 |
| Node.js | ✅ | — | ✅ | — | 2/4 |
| Docker / Kubernetes | — | — | ✅ | ✅ | 2/4 |
| **IA en el ciclo de desarrollo** | — | ✅ | ✅ | — | 2/4 |
| Python | — | ✅ | ✅ | — | 2/4 |
| SonarQube / Fortify | — | — | ✅ | ✅ | 2/4 |
| Diagramas (Visio / Draw.io) | — | — | ✅ | ✅ | 2/4 |
| Angular | — | ✅ | — | — | 1/4 |
| **Oracle / PL-SQL** | — | — | — | — | **0/4** |

## Las tres conclusiones

**1. El núcleo no es un lenguaje.** Lo que aparece en los cuatro es **API + arquitectura + entrega + colaboración**. Java, Node y .NET son intercambiables entre estos puestos; lo que no se negocia es saber diseñar, probar, desplegar y operar.

**2. Los cuatro piden diseñar + construir + operar.** Ninguno busca alguien que solo implemente tickets. Todos mencionan calidad, seguridad, resiliencia y diagnóstico. Es un perfil de **ciclo completo**, no de codificador.

**3. Oracle no aparece en ninguno.** Estaba en 4 de las 15 vacantes del radar, pero en **cero** de los puestos con peso. Es el ejemplo más claro de por qué cambiar el criterio cambia el plan.

---

# Qué cambia en el plan

## 🔺 Sube

| Tema | Por qué | Dónde |
|---|---|---|
| **Apéndice AWS** | Deja de ser opcional: 3 de 4 piden cloud práctico, y dos piden **AWS y Azure** | S13-14 |
| **ADRs y diagramas de arquitectura** | Es la entrega concreta de los dos puestos de arquitecto, y lo que valida "diseñar soluciones" | S11 |
| **Discurso de IA en el desarrollo** | 2 de 4 lo piden explícito, y ya lo hacés a diario | S14 |
| **Testing con TDD/BDD** | 3 de 4; TCS lo pide como BDD/TDD, no solo "tests" | S4, S8 |
| **SonarQube en el pipeline** | 2 de 4; es una línea de YAML | S13 |
| **Draw.io / diagramas** | 2 de 4; una tarde de trabajo | S11 |

## 🔻 Baja

| Tema | Por qué |
|---|---|
| **Oracle / PL-SQL** | 0 de 4 puestos con peso. Pasa de una semana propia a "solo si una candidatura concreta lo pide". **Libera la semana 10** |
| **Angular** | 1 de 4. Se mantiene el repaso corto de S14, no crece |
| **Kubernetes profundo** | 2 de 4, y como "deseable". Alcanza con fundamentos y diagnóstico |
| **Migración legacy / VB6 / WinForms** | 0 de 4. Fuera del plan |

## Qué hacer con la semana 10 liberada

Tres opciones, en orden de retorno para estos puestos:

1. **Reforzar el apéndice AWS** — pasar de "una Lambda" a un flujo serverless completo con SQS + EventBridge. Es el hueco más caro de los cuatro puestos.
2. **Python básico** — 2 de 4 lo piden como deseable. Con tu base, una semana alcanza para leer, modificar y automatizar; no para presentarte como desarrollador Python.
3. **Profundizar arquitectura aplicada** — más ADRs, más diagramas, un caso de diseño end-to-end escrito.

Recomendación: **la 1**. AWS es el único requisito duro de los cuatro que hoy tenés en cero.

---

# Honestidad sobre el alcance

No todos los puestos con peso son alcanzables en diciembre, y conviene tenerlo claro desde ahora:

| Puesto | ¿Diciembre 2026? | Qué falta |
|---|---|---|
| **Cuidafarma** (Node + AWS) | 🟡 **Posible** | AWS práctico + evidencia Node desplegado. Ambos cubiertos por el plan |
| **Synopsis** Senior FullStack | 🟡 **Posible** | Java demostrable (el laboratorio) + Python básico |
| **TCS** Software Architect | 🔴 No | Pide 5+ años **de arquitectura**. Es el arco de 2-3 años |
| **Stefanini** Solution Architect | 🔴 No | Pide 6 años dev + experiencia progresiva en arquitectura |

**Los dos primeros son el objetivo real de diciembre.** Los dos de arquitecto son el norte de [[08 - Norte de Carrera - Camino a Solution Architect]] — y lo importante es que el mismo plan construye hacia los cuatro: no son caminos distintos, es el mismo camino en dos etapas.

## La forma correcta de leer esto

```
2026 dic  →  Senior Backend que diseña, prueba, despliega y opera
                    (Cuidafarma / Synopsis / los mejores de la lista)
                              ↓
2028-29   →  Solution Architect
                    (TCS / Stefanini)
```

Lo que te falta para el primer nivel es **evidencia**: el laboratorio, AWS, el CV reescrito. Lo que te falta para el segundo es **tiempo ejerciendo decisiones** — y eso empieza en el trabajo que consigas en diciembre, no en un curso.

> [!important] El criterio que ordena todo de ahora en más
> Ante cualquier tema nuevo que aparezca: **¿lo piden 2 o más de los cuatro puestos con peso?**
>
> Si sí, entra al plan. Si no, se registra y se estudia solo cuando una candidatura concreta lo pida. Este filtro reemplaza al de frecuencia del radar.
