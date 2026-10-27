---
tags:
  - Java
  - Arquitectura
  - Entrevistas
Creado: 2026-09-08
Relacionado:
  - Java
  - Sistemas Distribuidos
  - Senior Java Engineer
Categoria: Senioridad
Estado:
  - Pendiente
Progreso: 0
Prioridad: Media-Alta
Nivel: Avanzado
Prerequisitos: Sistemas Distribuidos, Arquitectura Java, Performance Java
---

#Principal

## 🧠 Por qué existe este módulo

Es el módulo de cierre del roadmap. No enseña tecnología nueva: enseña a **usar todo lo anterior bajo presión y en voz alta**, que es exactamente lo que evalúa una entrevista senior.

Un candidato puede saber Kafka, Redis y hexagonal por separado y aun así fallar la entrevista de diseño, porque la habilidad que se mide es otra: acotar un problema ambiguo, estimar, elegir con criterio y **defender el trade-off** que elegiste.

## 🎯 Objetivos de Aprendizaje

- [ ] Conducir una entrevista de diseño de 45 minutos sin quedarte en blanco.
- [ ] Estimar volumen, almacenamiento y ancho de banda con números redondos.
- [ ] Justificar cada decisión con su costo, no solo con su beneficio.
- [ ] Identificar el cuello de botella antes de que te lo señalen.
- [ ] Diseñar los casos bancarios típicos de entrevista en el sector.

## 🧠 El método — 6 pasos, siempre los mismos

```
1. ACOTAR      · ¿Qué hace y qué NO hace? Escribí los requisitos funcionales.
                 Nunca empieces a dibujar antes de este paso.
2. ESCALA      · Usuarios, RPS, lectura vs escritura, tamaño del dato, retención.
                 Números redondos y en voz alta.
3. API         · Los 3-5 endpoints centrales. Fija el contrato antes que la infra.
4. DATOS       · Modelo, motor y por qué. Clave de partición si hay sharding.
5. DISEÑO      · Diagrama de alto nivel. Recién acá aparecen las cajas.
6. PROFUNDIZAR · El entrevistador elige un punto. Ahí demostrás profundidad:
                 cuello de botella, caché, consistencia, fallos, costo.
```

> [!tip] El error más común
> Saltar directo al paso 5. Si dibujás cajas antes de acotar y estimar, estás resolviendo un problema que nadie te pidió — y se nota.

## 📚 Temas

| #  | Tema | Estado | Nota |
|----|------|--------|------|
| 1  | El método de 6 pasos en detalle | ⬜ | [[01 - Metodo de Diseno]] |
| 2  | Estimaciones y capacity planning | ⬜ | [[02 - Estimaciones]] |
| 3  | Diseño de APIs a escala | ⬜ | [[03 - APIs a Escala]] |
| 4  | Caché — estrategias e invalidación | ⬜ | [[04 - Estrategias de Cache]] |
| 5  | Base de datos a escala — réplicas y sharding | ⬜ | [[05 - Base de Datos a Escala]] |
| 6  | Colas, backpressure y control de carga | ⬜ | [[06 - Colas y Backpressure]] |
| 7  | Rate limiting y throttling | ⬜ | [[07 - Rate Limiting]] |
| 8  | Alta disponibilidad y degradación elegante | ⬜ | [[08 - Alta Disponibilidad]] |
| 9  | Trade-offs y cómo defenderlos | ⬜ | [[09 - Tradeoffs]] |

## 🎯 Casos de práctica

### Genéricos (calientan la mano)

| # | Caso | Qué ejercita | Estado | Nota |
|---|------|--------------|--------|------|
| 1 | Acortador de URLs | hashing, caché, lectura masiva | ⬜ | [[C1 - Acortador de URLs]] |
| 2 | Feed de novedades | fan-out, denormalización | ⬜ | [[C2 - Feed de Novedades]] |
| 3 | Chat en tiempo real | WebSockets, presencia, orden | ⬜ | [[C3 - Chat en Tiempo Real]] |
| 4 | Carga masiva de archivos | streaming, batch, idempotencia | ⬜ | [[C4 - Carga Masiva de Archivos]] |

### Bancarios (los que importan para tu objetivo)

| # | Caso | Qué ejercita | Estado | Nota |
|---|------|--------------|--------|------|
| 5 | Motor de transferencias | consistencia, idempotencia, ledger | ⬜ | [[C5 - Motor de Transferencias]] |
| 6 | Procesador de pagos con tarjeta | autorización, timeouts, reversas | ⬜ | [[C6 - Procesador de Pagos]] |
| 7 | Estado de cuenta histórico | event sourcing, proyecciones | ⬜ | [[C7 - Estado de Cuenta]] |
| 8 | Motor de límites y antifraude | reglas, latencia, tiempo real | ⬜ | [[C8 - Limites y Antifraude]] |
| 9 | Cierre de día y conciliación | batch, cuadre, ventanas | ⬜ | [[C9 - Cierre de Dia]] |

## 🔗 Relaciones
- **Prerequisitos**: [[Sistemas Distribuidos]], [[Arquitectura Java]], [[Performance Java]]
- **Se conecta con**: [[Banca y Fintech]], [[Cloud para Java]], [[Senior Java Engineer]]
- **Evidencia práctica**: [[Proyectos Java]]

## 📌 Notas Rápidas
- En diseño no hay respuesta correcta, hay respuestas justificadas. "Depende" es válido **solo** si decís de qué depende.
- Practicá en voz alta y con reloj. Saber el tema y poder explicarlo en 45 minutos son habilidades distintas.
- Los casos bancarios son los que te diferencian: cualquiera prepara el acortador de URLs.

---

## ✍️ Mis apuntes
> 📓 Apuntes del módulo → [[Apuntes - System Design]]

<!-- Anotaciones rápidas sobre el módulo en general -->

