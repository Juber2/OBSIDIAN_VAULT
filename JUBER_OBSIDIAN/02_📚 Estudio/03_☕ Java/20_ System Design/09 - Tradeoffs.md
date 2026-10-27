---
tags:
  - Java
  - System Design
  - Entrevistas
Creado: 2026-09-08
Relacionado:
  - System Design
  - 08 - ADR y Tradeoffs
Palabras clave:
  - tradeoff, decision, ADR, puerta de ida, reversible, depende, justificar
---

## 🧠 Concepto

En diseño de sistemas **no hay respuestas correctas, hay respuestas justificadas**. Toda decisión mejora algo y empeora otra cosa; la habilidad que se evalúa es saber *qué* estás empeorando y *por qué* aceptás ese costo.

> [!danger] La señal más clara de inexperiencia
> Presentar una decisión solo por sus beneficios. *"Uso Kafka porque escala y desacopla"* deja afuera la mitad de la conversación: también agrega un sistema que operar, consistencia eventual, complejidad de depuración y una dependencia más en el camino crítico.
> **Una decisión sin costo declarado no es una decisión: es una preferencia.**

## 🎚️ Los ejes clásicos

| Eje | Ganás | Perdés |
|---|---|---|
| Consistencia ↔ Disponibilidad | datos correctos | responder durante una partición |
| Latencia ↔ Rendimiento | respuesta rápida | operaciones por segundo |
| Latencia ↔ Consistencia | lectura de réplica | datos actualizados |
| Simplicidad ↔ Flexibilidad | fácil de entender | casos de uso futuros |
| Costo ↔ Rendimiento | factura baja | capacidad de pico |
| Acoplamiento ↔ Autonomía | evolucionar por separado | consistencia y trazabilidad |
| Normalización ↔ Velocidad de lectura | sin duplicación | consultas más caras |
| Comprar ↔ Construir | tiempo de entrega | control y costo a largo plazo |

## 🗣️ Cómo se justifica una decisión

La estructura que funciona, en cuatro partes:

```
1 · QUÉ elegí
2 · POR QUÉ, atado a un requisito concreto
3 · QUÉ COSTO tiene
4 · CUÁNDO cambiaría de opinión
```

```
✅ "Uso PostgreSQL con bloqueo pesimista para las transferencias.

    Porque el requisito no funcional dice consistencia fuerte en el saldo:
    un doble gasto es inaceptable.

    El costo es que la contención sobre una cuenta muy activa serializa
    las operaciones, y limita el rendimiento de escritura de esa cuenta.

    Si apareciera un cliente corporativo con mil transferencias por minuto,
    cambiaría a un modelo de reservas o particionaría por sub-cuenta."
```

> [!tip] El punto 4 es el que más impresiona
> *"Cambiaría de opinión si..."* demuestra que la decisión no es dogmática: entendés el rango donde es válida y sabés cuál es la señal de que dejó de serlo.
> Es la diferencia entre alguien que aprendió una solución y alguien que entiende el problema.

## 🚪 Puertas de ida y de vuelta

La clasificación más útil para decidir cuánto tiempo dedicarle a una decisión:

| | Puerta de vuelta | Puerta de ida |
|---|---|---|
| Reversible | ✅ barato | ❌ muy caro |
| Cuánto pensarla | poco: probá y ajustá | mucho: analizá bien |
| Ejemplos | biblioteca de logs, formato de logs, tamaño de instancia, nombre de un endpoint interno | clave de partición, contrato de API pública, motor de base, modelo de datos, formato de eventos publicados |

> [!warning] Tratar todas las decisiones como puertas de ida paraliza
> Discutir tres días qué biblioteca de logs usar es tiempo perdido: se cambia en una tarde. Ese tiempo hay que gastarlo en la clave de partición, que es una migración de meses.
> **Identificá primero de qué tipo es la decisión, y ahí decidí cuánto analizarla.**

```
¿Cuánto cuesta revertir esto en 6 meses?
├── una tarde        → decidí rápido, probá
├── un sprint        → una reunión, un ADR corto
└── meses / imposible → analizá en serio, prototipá, escribí el ADR completo
```

## ⚖️ "Depende" bien usado

```
❌ "Depende."
❌ "Depende del caso de uso."

✅ "Depende de si toleramos consistencia eventual en la lectura del saldo.
    Si el saldo que muestro puede tener 2 segundos de atraso, uso réplicas
    y escalo lecturas linealmente.
    Si tiene que ser exacto, todo va a la primaria y necesito otra estrategia.
    ¿Qué asumimos?"
```

> [!tip] "Depende" solo vale si nombrás la variable y las dos ramas
> Y termina en una pregunta o en una asunción explícita. Si no, es una forma elegante de no responder — y el entrevistador lo nota.

## 🧮 Cuantificar cuando se pueda

```
"Redis mejora la latencia"                    ← opinión
"Redis baja el p99 de 180 ms a 12 ms,
 con un acierto del 92%, por 40 USD/mes"      ← argumento
```

| Dimensión | Cómo se mide |
|---|---|
| Latencia | p50, p95, p99 |
| Rendimiento | peticiones por segundo sostenidas |
| Costo | USD por mes, o por transacción |
| Disponibilidad | nueves, minutos de caída |
| Complejidad | componentes nuevos que operar |
| Tiempo de entrega | semanas de desarrollo |

> [!tip] La complejidad se cuantifica contando piezas
> *"Esto agrega un servicio más que desplegar, monitorear, respaldar y actualizar, y un modo de fallo nuevo"* es un costo concreto. Decir "es más complejo" sin más es una impresión.

## 📋 Cuatro decisiones típicas, con sus dos caras

### Monolito vs microservicios

| Monolito modular | Microservicios |
|---|---|
| ✅ transacciones locales, despliegue simple, depuración directa | ✅ despliegue y escalado independientes, equipos autónomos |
| ❌ escalado conjunto, despliegue acoplado | ❌ consistencia eventual, latencia de red, operación compleja |

> [!warning] Microservicios resuelve un problema organizativo, no técnico
> Si tenés un equipo de cinco personas, la autonomía de despliegue no te compra nada y la complejidad te cuesta todo. La respuesta honesta para la mayoría de los sistemas es **monolito modular** — ver [[07 - Modular Monolith]].

### SQL vs NoSQL

| Relacional | NoSQL clave-valor |
|---|---|
| ✅ transacciones, `JOIN`, integridad, consultas flexibles | ✅ escala horizontal, latencia predecible, esquema flexible |
| ❌ escala vertical primero, esquema rígido | ❌ sin transacciones entre entidades, consultas limitadas al patrón de acceso |

> [!tip] La pregunta correcta es por el patrón de acceso, no por el volumen
> "Tenemos muchos datos" no justifica NoSQL: PostgreSQL maneja terabytes. Lo que lo justifica es **acceso por clave, latencia predecible y escala horizontal**. Si tu dominio tiene relaciones y consultas variadas, relacional gana aunque haya volumen.

### Sincrónico vs asincrónico

| Síncrono | Asíncrono |
|---|---|
| ✅ resultado inmediato, errores claros, simple de depurar | ✅ desacopla, absorbe picos, tolera caídas |
| ❌ acopla disponibilidad, latencia acumulada | ❌ consistencia eventual, más difícil de seguir |

### Comprar vs construir

| Comprar | Construir |
|---|---|
| ✅ rápido, mantenido por otro | ✅ control total, sin costo de licencia |
| ❌ costo recurrente, dependencia, límites ajenos | ❌ tiempo de desarrollo y mantenimiento perpetuo |

> [!tip] Construí lo que te diferencia, comprá el resto
> Un banco construye su motor de transferencias y compra el servicio de correo. Al revés no tendría sentido: nadie elige un banco por su servidor de correo.

## 📝 Registrar la decisión

```markdown
# ADR-012: bloqueo pesimista para transferencias

## Estado
Aceptada — 2026-09-08

## Contexto
Las transferencias deben garantizar que no se pueda gastar dos veces.
La contención sobre una misma cuenta es realista (cuentas corporativas).

## Decisión
Bloqueo pesimista (`SELECT FOR UPDATE`) con orden determinista por UUID.

## Consecuencias
### Positivas
- Elimina la condición de carrera del saldo.
- El orden determinista hace imposible el deadlock.

### Negativas
- Serializa las operaciones sobre una misma cuenta.
- Requiere transacciones cortas: prohibido llamar a servicios externos con el bloqueo tomado.

## Alternativas descartadas
- **Bloqueo optimista**: con alta contención, demasiados reintentos.
- **SERIALIZABLE**: más simple de escribir, peor rendimiento y abortos frecuentes.

## Cuándo revisar
Si el p99 de transferencias supera 1 s por contención sobre una sola cuenta.
```

> [!tip] La sección "cuándo revisar" es la que más se olvida y más vale
> Convierte el documento en algo vivo: define la señal concreta que indica que la decisión dejó de ser válida. Sin eso, los ADR se leen una vez y se archivan.

Detalle en [[09 - ADR y Documentacion]].

## 🎤 En la entrevista

| Situación | Qué decir |
|---|---|
| Te dan a elegir | *"Elijo X. Gano A, pierdo B. Con estos requisitos, A pesa más."* |
| Te objetan | *"Buen punto. Si eso es cierto, la decisión cambia: iría por Y."* |
| No hay datos | *"Depende de Z. Asumo que Z es así; si no, cambia."* |
| Todo parece igual | *"Son equivalentes técnicamente; elijo la más simple de operar."* |
| Te preguntan por qué no otra cosa | *"Lo consideré. Lo descarté porque..."* |

> [!danger] Aferrarse a una decisión mala es peor que el error inicial
> Los entrevistadores objetan a propósito para ver cómo reaccionás. Cambiar de opinión ante un argumento válido —diciéndolo en voz alta— es una señal positiva, no una derrota.
> Lo que sí resta es cambiar de opinión **sin argumento**, solo porque el entrevistador puso cara rara.

## ✅ Antes de cerrar un diseño

- [ ] ¿Cada decisión importante tiene su porqué atado a un requisito?
- [ ] ¿Dije el costo de cada una, no solo el beneficio?
- [ ] ¿Identifiqué cuáles son puertas de ida?
- [ ] ¿Mencioné qué alternativas descarté y por qué?
- [ ] ¿Dije bajo qué condición cambiaría de opinión?
- [ ] ¿Señalé el punto más frágil del diseño?
- [ ] ¿Elegí la opción más simple que cumple los requisitos?

## ⚠️ Errores Comunes

> [!warning] Los que se notan enseguida
> - **Presentar solo beneficios.**
> - **"Depende" sin decir de qué.**
> - **Elegir por moda** en vez de por requisito.
> - **Tratar toda decisión como irreversible**: parálisis.
> - **Tratar una puerta de ida como reversible**: se paga meses después.
> - **No mencionar alternativas descartadas.**
> - **Defender lo indefendible** ante una objeción válida.
> - **Complejidad sin justificación**: la simplicidad es el default.
> - **No registrar el porqué**: en seis meses nadie recuerda la razón.

## 📌 Notas Rápidas
- La frase que resume el módulo entero: **toda decisión tiene un costo; si no podés nombrarlo, no entendiste la decisión**.
- La clasificación puerta de ida / puerta de vuelta es lo más aplicable al trabajo diario: te dice dónde invertir el tiempo de análisis y dónde simplemente probar.
- En una entrevista, decir *"la opción más simple que cumple los requisitos"* como criterio de desempate es una respuesta fuerte. La complejidad hay que justificarla; la simplicidad es el punto de partida.
- Este es el cierre del módulo: los casos de práctica ([[C1 - Acortador de URLs]] a [[C9 - Cierre de Dia]]) son donde se ejercita todo esto junto.

## 🔗 Relaciones
- **Módulo**: [[System Design]] — última nota teórica del módulo
- **Anterior**: [[08 - Alta Disponibilidad]]
- **Se conecta con**: [[08 - ADR y Tradeoffs]], [[01 - Metodo de Diseno]], [[Senior Java Engineer]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

