---
tags:
  - Estudio
  - Capacitación
  - Roadmap
Creado: 2026-08-31
Actualizado: 2026-08-31
Inicio: 2026-09-01
Fin: 2026-12-31
Relacionado:
  - PLAN DE CAPACITACION
  - 09 - Perfil Objetivo - Puestos con Peso
---

# Mapa de capacitación — qué aprender y qué reforzar

Qué estudiar de septiembre a diciembre de 2026, derivado de **los 5 avisos completos** que definen el objetivo. No de una lista genérica de "lo que se usa hoy".

## Las 5 fuentes

| Aviso | Perfil | Señal |
|---|---|---|
| **Cuidafarma** (Grupo Breca) | Backend Node + AWS | S/ 6.500-7.500 · pide 3 años · acepta título técnico |
| **Synopsis** | Senior FullStack Java + Angular + Python + IA | S/ 4.000-7.000 |
| **TCS Perú** | Software Architect Full Stack | La vara más alta |
| **Stefanini** | Solution Architect | Norte de carrera |
| **Coforge / BCP** | Senior Java Developer | Banca, híbrido Lima |

## Lo que el mercado pide, contado sobre esos 5

| Capacidad | Lo piden | |
|---|:-:|---|
| **APIs REST + OpenAPI/Swagger** | **5/5** | Ninguno prescinde de esto |
| **Clean Architecture / SOLID / DDD** | **5/5** | Aparece hasta en el aviso de Node |
| **Microservicios** | 4/5 | |
| **Cloud práctico** | 4/5 | AWS en 3, Azure en 3, ambos exigidos juntos en 1 |
| **CI/CD** | 4/5 | |
| **Observabilidad y diagnóstico** | 4/5 | |
| **Event-driven, colas, idempotencia, DLQ** | 4/5 | |
| **Git y trabajo colaborativo** | 4/5 | |
| Seguridad: OAuth2, JWT, OWASP | 3/5 | |
| Testing (unit, integración, TDD/BDD) | 3/5 | |
| Docker y Kubernetes | 3/5 | |
| Java + Spring Boot | 3/5 | |
| Bases relacionales y SQL | 3/5 | |
| Node.js | 2/5 | |
| Python | 2/5 | |
| Quarkus | 2/5 | |
| IA en el ciclo de desarrollo | 2/5 | |
| SonarQube / análisis estático | 2/5 | |
| Diagramas de arquitectura | 2/5 | |
| Angular · Microfrontends · FinOps · Postman | 1/5 c/u | |

> **La conclusión que ordena todo:** lo que aparece en 4 o 5 de 5 **no es un lenguaje**. Es API + arquitectura + entrega + operación. Java, Node y .NET son intercambiables entre estos puestos; lo que no se negocia es saber diseñar, probar, desplegar y operar.

---

# ✅ Ya lo tenés — no invertir tiempo, solo documentarlo

Esto ya lo hacés con experiencia real. El trabajo acá **no es estudiar: es que aparezca en el CV** (ver [[24 - Guia de Adaptacion del CV (.NET y Java)]]).

| Capacidad | Piden | Tu evidencia |
|---|:-:|---|
| APIs REST | 5/5 | 5 años construyéndolas en 4 empresas |
| Clean Architecture, SOLID, **CQRS** | 5/5 | Pluxee + sistema de capacitación (MediatR) |
| Bases relacionales, SQL, índices, procedures | 3/5 | 5 años · optimización con planes de ejecución |
| CI/CD | 4/5 | Armaste el pipeline YAML en Azure DevOps |
| Git | 4/5 | Trabajo diario |
| Docker | 3/5 | Contenerización en Pluxee |
| Integraciones con terceros | 4/5 | Izipay · APIs para apps móviles |
| Procesos batch y ETL | 2/5 | Pluxee, alto volumen |
| Migración legacy | — | Pluxee, back office |

---

# 🔧 Reforzar — lo sabés en teoría, falta profundidad y evidencia

Hay notas en el vault, pero **nada en el CV y nada desplegado**. Este bloque es el grueso del plan.

| Capacidad | Piden | Qué falta exactamente | Semanas |
|---|:-:|---|---|
| **Microservicios** | 4/5 | Un sistema real con más de un servicio | S3, S9 |
| **Event-driven: idempotencia, Outbox, DLQ** | 4/5 | Haberlo implementado y probado el duplicado | S9 |
| **Observabilidad** | 4/5 | Logs estructurados, métricas, trazas correlacionadas | S12 |
| **Testing** | 3/5 | **Cero menciones en tu CV.** Unit, integración, Testcontainers | S4, S8 |
| **Seguridad OAuth2/JWT/OWASP** | 3/5 | Implementado, no leído. Autorización por recurso | S3, S7 |
| **DDD y arquitectura hexagonal** | 3/5 | Un caso aplicado, no definiciones | S1, S11 |
| **Azure a nivel servicios** | 3/5 | Tenés migración; falta Key Vault, Service Bus, App Insights | S2, S12 |
| **Kubernetes** | 3/5 | Módulos 1-6 hechos; falta desplegar y diagnosticar | S12 |
| Documentación: OpenAPI, ADR, diagramas | 4/5 | Swagger sí; ADR y diagramas formales no | S11 |

---

# 📚 Aprender — desde cero

| Capacidad | Piden | Estado | Prioridad |
|---|:-:|---|---|
| **AWS** — Lambda, API Gateway, S3, SQS, EventBridge, RDS, CloudWatch | **3/5** | 🔴 **Cero.** Las 17 apariciones de "Lambda" en el vault son de Java | 🔺 **Máxima** |
| **Java + Spring Boot** | 3/5 | 163 notas de contenido, cero experiencia | 🔺 Alta |
| **Análisis estático (SonarQube)** | 2/5 | Cero. Es 1 línea de YAML | 🔹 Barata |
| **Diagramas formales (Draw.io)** | 2/5 | Cero. Una tarde | 🔹 Barata |
| **Discurso de IA en el desarrollo** | 2/5 | Ya lo hacés a diario, falta ordenarlo | 🔹 **La más barata** |
| **Agile / Scrum** | — | Vocabulario. Medio día | 🔹 Barata |
| Python | 2/5 | Cero | 🔸 Media |
| Quarkus | 2/5 | 9 notas, sin práctica | 🔸 Baja — 2 días desde Spring |
| Node.js desplegado | 2/5 | 89 notas de NestJS, sin evidencia | 🔸 Media |

> [!important] AWS es el único requisito duro que tenés en cero absoluto
> Tres de los cinco avisos lo piden, dos de ellos como **experiencia práctica**, y uno exige **AWS y Azure juntos**. Como ya tenés Azure real, sumar AWS te vuelve candidato **multi-cloud** — categoría rara. Saber Java y .NET, en cambio, es común.

---

# 🚫 Fuera del plan

| Tema | Por qué |
|---|---|
| **Oracle / PL-SQL** | 0 de los 5. Tres días cuando una entrevista lo pida |
| **Angular** | 1 de 5, y ya lo sabés. Solo repaso en S14 |
| Microfrontends, FinOps | 1 de 5 y son especializaciones |
| ELK / Dynatrace / Kibana | Los conceptos son los mismos de App Insights y CloudWatch |
| ArgoCD, Prometheus, AZ-104, AZ-400 | Del roadmap DevOps 2025. Otro rol |
| NestJS más profundo | 89 notas ya escritas. Falta evidencia, no contenido |
| Certificaciones | Vuelven en 2027: **AWS SAA** y **AZ-305**. Después de la experiencia, no antes |

---

# Cómo se reparten los 4 meses

| Mes | Foco | Por qué ahí |
|---|---|---|
| **Septiembre** | Arquitectura y patrones · datos · testing · Git/Docker/Scrum | Arquitectura está en 5/5 y no depende de nada. Testing es el hueco más visible del CV |
| **Octubre** | El laboratorio: microservicios, seguridad, mensajería, idempotencia | Convierte 4 capacidades de "🔧 reforzar" en evidencia desplegada |
| **Noviembre** | **AWS práctico** · observabilidad · CI/CD · ADRs y diagramas | AWS es el 🔴 más caro. Los ADRs son la credencial de diseño |
| **Diciembre** | CV, simulacros, postular | Sin temas nuevos |

## El criterio para lo que aparezca de acá en adelante

> **¿Lo piden 2 o más de los 5 avisos?**
>
> Si sí, entra al plan. Si no, se registra y se estudia solo cuando una candidatura concreta lo pida.

## La medida de que funcionó

En diciembre, poder decir las tres:

1. *"Diseñé, construí y desplegué un sistema con varios servicios, en dos nubes."*
2. *"Puedo explicar cada decisión y cuál fue la alternativa que descarté."*
3. *"Está corriendo, acá está la URL y el repo."*

Eso responde, sin adornos, lo que los cinco avisos preguntan de distinta forma.
