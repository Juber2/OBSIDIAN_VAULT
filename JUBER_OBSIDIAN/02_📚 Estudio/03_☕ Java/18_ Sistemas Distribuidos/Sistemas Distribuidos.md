---
tags:
  - Java
  - Distribuido
  - Arquitectura
Creado: 2026-09-08
Relacionado:
  - Java
  - Microservicios
  - Banca y Fintech
Categoria: Arquitectura
Estado:
  - Pendiente
Progreso: 0
Prioridad: Alta
Nivel: Avanzado
Prerequisitos: Microservicios, Mensajeria, Arquitectura Java
---

#Principal

## 🧠 Por qué existe este módulo

[[Microservicios]] enseña **cómo partir un sistema y comunicarlo**: Gateway, Feign, Eureka, Circuit Breaker. Este módulo enseña **qué se rompe cuando lo partís**, que es una pregunta distinta y bastante más difícil.

En cuanto hay dos procesos y una red entre ellos, dejan de existir garantías que dabas por sentadas: que un mensaje llega una sola vez, que dos nodos ven el mismo estado, que el reloj de A y el de B coinciden. Este módulo es el que separa "sé usar microservicios" de "sé diseñar sistemas distribuidos".

## 🎯 Objetivos de Aprendizaje

- [ ] Explicar qué garantía se pierde al distribuir y qué se gana a cambio.
- [ ] Hacer idempotente cualquier operación que cruce la red.
- [ ] Implementar Transactional Outbox y saber por qué el dual-write está roto.
- [ ] Elegir entre SAGA orquestada y coreografiada con criterio, no por moda.
- [ ] Reconocer cuándo Event Sourcing resuelve un problema y cuándo lo crea.
- [ ] Razonar sobre orden causal sin depender de timestamps.

## 🧠 Conceptos Fundamentales

- **La red falla, y falla de la peor forma**: no con un error, sino con un timeout. No sabés si la operación se ejecutó.
- **`at-least-once` es lo que hay.** `exactly-once` no existe a nivel de transporte: se construye con idempotencia del lado del consumidor.
- **Dual-write es un bug, no un patrón.** Escribir en base y publicar en el broker no es atómico. Ahí nace el Outbox.
- **La consistencia eventual es una decisión de negocio**, no un detalle técnico: alguien tiene que aceptar ver un dato viejo por 2 segundos.

## 📚 Temas

| #  | Tema | Estado | Nota |
|----|------|--------|------|
| 1  | Falacias de la computación distribuida | ⬜ | [[01 - Falacias de la Computacion Distribuida]] |
| 2  | CAP, PACELC y modelos de consistencia | ⬜ | [[02 - CAP PACELC y Consistencia]] |
| 3  | Idempotencia y deduplicación | ⬜ | [[03 - Idempotencia y Deduplicacion]] |
| 4  | Entrega de mensajes — at-least/at-most/exactly once | ⬜ | [[04 - Garantias de Entrega]] |
| 5  | Transactional Outbox e Inbox | ⬜ | [[05 - Transactional Outbox]] |
| 6  | SAGA — orquestación vs coreografía | ⬜ | [[06 - SAGA]] |
| 7  | Compensación y reversas | ⬜ | [[07 - Compensacion y Reversas]] |
| 8  | Event Sourcing | ⬜ | [[08 - Event Sourcing]] |
| 9  | CQRS distribuido y proyecciones | ⬜ | [[09 - CQRS Distribuido]] |
| 10 | Concurrencia distribuida — locks y versionado | ⬜ | [[10 - Locks y Bloqueo Optimista]] |
| 11 | Relojes, orden y causalidad | ⬜ | [[11 - Relojes y Causalidad]] |
| 12 | Particionado y sharding | ⬜ | [[12 - Particionado y Sharding]] |
| 13 | Consenso — quórum, líder y Raft | ⬜ | [[13 - Consenso y Raft]] |
| 14 | Multi-región, DR, RPO y RTO | ⬜ | [[14 - Multiregion y DR]] |

## 🔀 Árbol de decisión rápido

```
¿La operación cruza un límite de proceso?
├── NO  → transacción ACID local. No compliques.
└── SÍ  → ¿necesita respuesta inmediata?
         ├── SÍ → llamada síncrona + timeout + Circuit Breaker + idempotencia
         └── NO → evento asíncrono
                  ├── ¿hay que escribir en base Y publicar? → Outbox (nunca dual-write)
                  └── ¿varios pasos que pueden fallar a la mitad? → SAGA + compensación
```

## ⚠️ Errores clásicos

> [!warning] Los que más se ven en producción
> - **Dual-write**: `repo.save(x); broker.publish(e);` — si el segundo falla, quedaste inconsistente para siempre.
> - **Retry sin idempotencia**: multiplica el efecto en vez de repararlo.
> - **Ordenar por timestamp** entre nodos distintos: los relojes derivan, el orden que ves es mentira.
> - **SAGA sin compensación probada**: el camino feliz anda; el de reversa nunca se testeó.

## 🔗 Relaciones
- **Prerequisitos**: [[Microservicios]], [[Mensajeria]], [[Arquitectura Java]]
- **Se conecta con**: [[Banca y Fintech]], [[Cloud para Java]], [[System Design]]
- **Se aplica en**: [[Fase 5 - Eventos y Outbox]], [[Fase 6 - Microservicios y SAGA]]

## 📌 Notas Rápidas
- Casi toda pregunta de entrevista senior sobre microservicios es en realidad una pregunta de este módulo disfrazada.
- Si podés resolver el problema sin distribuir, hacelo: un monolito modular bien hecho gana casi siempre — ver [[07 - Modular Monolith]].

---

## ✍️ Mis apuntes
> 📓 Apuntes del módulo → [[Apuntes - Sistemas Distribuidos]]

<!-- Anotaciones rápidas sobre el módulo en general -->

