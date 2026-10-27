---
tags:
  - Java
  - Banca
  - Dominio
Creado: 2026-09-08
Relacionado:
  - Java
  - Sistemas Distribuidos
  - Proyectos Java
Categoria: Dominio
Estado:
  - Pendiente
Progreso: 0
Prioridad: Alta
Nivel: Intermedio-Avanzado
Prerequisitos: Spring Boot, Spring Data, Arquitectura Java
---

#Principal

## 🧠 Por qué existe este módulo

Todos los demás módulos enseñan **herramienta**. Este enseña **dominio**.

La diferencia entre un backend senior genérico y uno que entra a un banco no es que sepa más Spring: es que entiende por qué una transferencia no puede ejecutarse dos veces, por qué el saldo no se guarda como `double`, y qué significa que a fin de día tenga que cuadrar.

Es el módulo que convierte el resto del roadmap en un perfil contratable en banca, fintech o medios de pago.

## 🎯 Objetivos de Aprendizaje

- [ ] Representar dinero sin perder un centavo, en cualquier moneda.
- [ ] Diseñar un ledger de partida doble que siempre cuadre.
- [ ] Garantizar que una operación financiera se ejecute exactamente una vez.
- [ ] Explicar el ciclo de vida de un pago con tarjeta y de una transferencia interbancaria.
- [ ] Diseñar la traza de auditoría que exige un regulador.
- [ ] Reconocer qué obligaciones impone PCI-DSS sobre el código.

## 🧠 Conceptos Fundamentales

- **El saldo no es un campo, es un resultado.** Se deriva de los movimientos, no se guarda y pisa.
- **Nada se borra.** Un error no se corrige editando: se corrige con un asiento de reversa.
- **Idempotencia antes que retry.** Reintentar sin clave de idempotencia es duplicar plata.
- **Cuadrar es la prueba.** Si la suma de débitos no iguala la de créditos, el sistema está mal, aunque la API responda 200.

## 📚 Temas

| #  | Tema | Estado | Nota |
|----|------|--------|------|
| 1  | Dinero en software — BigDecimal, escala y redondeo | ⬜ | [[01 - Dinero en Software]] |
| 2  | Multi-moneda y tipos de cambio | ⬜ | [[02 - Multimoneda y Tipos de Cambio]] |
| 3  | Ledger y partida doble | ⬜ | [[03 - Ledger y Partida Doble]] |
| 4  | Cuentas, saldos y movimientos | ⬜ | [[04 - Cuentas Saldos y Movimientos]] |
| 5  | Idempotencia en operaciones financieras | ⬜ | [[05 - Idempotencia Financiera]] |
| 6  | Transferencias y consistencia | ⬜ | [[06 - Transferencias y Consistencia]] |
| 7  | Medios de pago — tarjetas e ISO 8583 | ⬜ | [[07 - Medios de Pago y ISO 8583]] |
| 8  | Mensajería financiera — ISO 20022 y SWIFT | ⬜ | [[08 - ISO 20022 y SWIFT]] |
| 9  | Conciliación y cuadre | ⬜ | [[09 - Conciliacion y Cuadre]] |
| 10 | Cierre de día y procesos batch | ⬜ | [[10 - Cierre de Dia y Batch]] |
| 11 | Seguridad y cumplimiento — PCI-DSS y HSM | ⬜ | [[11 - PCI-DSS y HSM]] |
| 12 | Auditoría, trazabilidad y no repudio | ⬜ | [[12 - Auditoria y Trazabilidad]] |
| 13 | Antifraude, límites y cupos | ⬜ | [[13 - Antifraude y Limites]] |
| 14 | Open Banking y APIs financieras | ⬜ | [[14 - Open Banking]] |

## 💼 Casos que se practican en el laboratorio

| Caso | Dónde se implementa |
|---|---|
| Alta de cuenta con validación de identidad | [[Fase 1 - API de Cuentas]] |
| Transferencia entre cuentas propias | [[Fase 2 - Transferencias e Idempotencia]] |
| Reverso de una operación errónea | [[Fase 2 - Transferencias e Idempotencia]] |
| Estado de cuenta y saldo histórico | [[Fase 4 - Hexagonal y Testing]] |
| Notificación de movimiento por evento | [[Fase 5 - Eventos y Outbox]] |
| Transferencia interbancaria con SAGA | [[Fase 6 - Microservicios y SAGA]] |
| Cierre de día y conciliación | [[Fase 8 - Batch y Cierre de Dia]] |

## ⚠️ Errores que cuestan dinero

> [!danger] Los cuatro clásicos
> 1. Importes en `double` → descuadres de centavos que aparecen recién en el cierre.
> 2. `POST` sin clave de idempotencia → el reintento del cliente duplica la transferencia.
> 3. Saldo como columna actualizable → dos requests concurrentes y el saldo queda mal.
> 4. Borrar o editar movimientos → se pierde la traza y el regulador lo rechaza.

## 🔗 Relaciones
- **Prerequisitos**: [[Spring Boot]], [[Spring Data]], [[Arquitectura Java]]
- **Se conecta con**: [[Sistemas Distribuidos]], [[Mensajeria]], [[Spring Security]]
- **Se aplica en**: [[Proyectos Java]]

## 📌 Notas Rápidas
- Este módulo no depende de Java: los conceptos valen igual en .NET. Lo que cambia es la implementación.
- Si en una entrevista de banca te preguntan "¿cómo evitás una transferencia duplicada?", la respuesta esperada es clave de idempotencia + restricción única en base, no "uso transacciones".

---

## ✍️ Mis apuntes
> 📓 Apuntes del módulo → [[Apuntes - Banca y Fintech]]

<!-- Anotaciones rápidas sobre el módulo en general -->

