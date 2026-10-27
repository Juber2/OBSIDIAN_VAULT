---
tags:
  - Java
  - Avanzado
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
Nivel: Avanzado
Prerequisitos: Java Intermedio
---

#Principal

## 📚 Temas

| #   | Tema                                  | Estado | Nota                                        |
| --- | ------------------------------------- | ------ | ------------------------------------------- |
| 1   | Concurrencia - Threads y Runnable     | ⬜      | [[01 - Concurrencia Threads y Runnable]]    |
| 2   | Concurrencia - ExecutorService        | ⬜      | [[02 - ExecutorService y Thread Pools]]     |
| 3   | CompletableFuture                     | ⬜      | [[03 - CompletableFuture]]                  |
| 4   | JVM - Memoria y Garbage Collection    | ⬜      | [[04 - JVM Memoria y GC]]                   |
| 5   | Reflection                            | ⬜      | [[05 - Reflection]]                         |
| 6   | Programación Funcional Avanzada       | ⬜      | [[06 - Programacion Funcional Avanzada]]    |
| 7   | Módulos JPMS                          | ⬜      | [[07 - Modulos JPMS]]                       |
| 8   | Virtual Threads (Java 21)             | ⬜      | [[08 - Virtual Threads]]                    |
| 9   | Modelo de Memoria de Java (JMM) 🆕    | ⬜      | [[09 - Modelo de Memoria de Java]]          |
| 10  | equals, hashCode e inmutabilidad 🆕   | ⬜      | [[10 - equals hashCode e Inmutabilidad]]    |
| 11  | Internals de colecciones 🆕           | ⬜      | [[11 - Internals de Colecciones]]           |
| 12  | Structured Concurrency 🆕             | ⬜      | [[12 - Structured Concurrency]]             |
| 13  | Pattern matching y sealed types 🆕    | ⬜      | [[13 - Pattern Matching Moderno]]           |

## 🧠 Por qué se agregaron los temas 9–13

Los temas 1–8 cubren **qué ofrece la plataforma**. Los nuevos cubren **por qué se comporta así**, que es lo que se pregunta en entrevistas senior:

- El **JMM** explica por qué un programa concurrente puede ver datos viejos aunque el código "parezca" correcto — y por qué `volatile` no es un candado.
- El contrato **`equals`/`hashCode`** es la causa raíz de bugs silenciosos en `HashMap` y `distinct()`.
- Los **internals de colecciones** son la diferencia entre elegir `ArrayList` por costumbre y elegirlo por complejidad algorítmica.
- **Structured Concurrency** (Java 21) es la evolución de `ExecutorService`: cancelación y propagación de errores por alcance.

## 🔗 Relaciones
- **Prerequisitos**: [[Java Intermedio]]
- **Siguiente paso**: [[Patrones de Diseño]], [[Testing en Java]]
- **Se conecta con**: [[Performance Java]], [[Sistemas Distribuidos]]

---

## ✍️ Mis apuntes
> 📓 Apuntes del módulo → [[Apuntes - Avanzado]]

<!-- Anotaciones rápidas sobre el módulo en general -->

