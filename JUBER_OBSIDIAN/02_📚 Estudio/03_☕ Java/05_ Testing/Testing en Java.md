---
tags:
  - Java
  - Testing
Creado: 2026-05-21
Relacionado:
  - Spring Boot
  - Java Avanzado
Nivel: Intermedio
---

#Principal

## 📚 Temas

| #  | Tema | Estado | Nota |
|----|------|--------|------|
| 1  | JUnit 5 | ⬜ | [[01 - JUnit 5]] |
| 2  | Mockito | ⬜ | [[02 - Mockito]] |
| 3  | TDD (Test Driven Development) | ⬜ | [[03 - TDD]] |
| 4  | Integration Testing | ⬜ | [[04 - Integration Testing]] |
| 5  | TestContainers | ⬜ | [[05 - TestContainers]] |
| 6  | Testing en Spring Boot | ⬜ | [[06 - Testing en Spring Boot]] |
| 7  | Testing de arquitectura con ArchUnit 🆕 | ⬜ | [[07 - ArchUnit]] |
| 8  | Contract testing con Pact 🆕 | ⬜ | [[08 - Contract Testing]] |
| 9  | Mutation testing con PIT 🆕 | ⬜ | [[09 - Mutation Testing]] |
| 10 | Pruebas de carga con Gatling y k6 🆕 | ⬜ | [[10 - Pruebas de Carga]] |

## 🧪 Qué prueba cada nivel

| Nivel | Responde a | Herramienta | Velocidad |
|---|---|---|---|
| Unidad | ¿la lógica es correcta? | JUnit + Mockito | ms |
| Integración | ¿los componentes se entienden entre sí? | `@SpringBootTest` + Testcontainers | segundos |
| Arquitectura | ¿alguien violó las reglas de capas? | ArchUnit | ms |
| Contrato | ¿el consumidor y el proveedor siguen compatibles? | Pact | segundos |
| Mutación | ¿mis tests **detectan** errores o solo ejecutan código? | PIT | minutos |
| Carga | ¿aguanta el volumen esperado? | Gatling / k6 | minutos |

> [!tip] Por qué importa el mutation testing
> La cobertura mide qué líneas se **ejecutan**, no qué errores se **detectan**. Un test sin asserts da 100% de cobertura y no prueba nada. PIT introduce fallos a propósito y verifica que algún test se rompa.

## 🔗 Relaciones
- **Prerequisitos**: [[Java Fundamentos]], [[Java Intermedio]]
- **Se conecta con**: [[Spring Boot]], [[Spring Data]], [[Arquitectura Java]]
- **Se aplica en**: [[Fase 4 - Hexagonal y Testing]]

---

## ✍️ Mis apuntes
> 📓 Apuntes del módulo → [[Apuntes - Testing]]

<!-- Anotaciones rápidas sobre el módulo en general -->

