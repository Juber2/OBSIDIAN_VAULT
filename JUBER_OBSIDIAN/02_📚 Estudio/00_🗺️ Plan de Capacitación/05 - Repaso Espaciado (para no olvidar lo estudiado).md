---
tags:
  - Postulaciones
  - Aprendizaje
  - Repaso
Creado: 2026-08-31
Relacionado:
  - 04 - Plan Detallado 3 Semanas (dia a dia)
  - 20 - Banco de Preguntas de las 14 Vacantes
---

# Repaso espaciado (para no olvidar lo estudiado)

## El problema que corrige esta nota

[[04 - Plan Detallado 3 Semanas (dia a dia)]] tiene un defecto de diseño: **toca cada tema una sola vez y avanza**. Arquitectura hexagonal se estudia el 31 de agosto y no vuelve a aparecer en todo el plan. Los domingos solo repasan lo marcado 🟡 y ❌ — lo que quedó ✅ se da por sabido y se abandona.

Pero lo marcado ✅ también se olvida. Sin repaso, un tema estudiado una vez se pierde en semanas. Si la entrevista cae en octubre, la mitad de lo que estudiaste en septiembre ya no está disponible cuando lo necesitás.

> [!warning] El error específico
> No es estudiar poco: es estudiar **una vez**. Un tema visto cuatro veces en intervalos crecientes queda disponible meses; el mismo tema visto una vez durante cuatro horas seguidas, no.

## La regla: repasar a los 3, 7 y 21 días

```
Día 0    →  Estudio activo (bloque 🌅 o 🌙)     — aprender
Día +3   →  Repaso corto (5-10 min)              — antes de que se borre
Día +7   →  Repaso corto (5-10 min)              — consolidar
Día +21  →  Repaso corto (5-10 min)              — fijar a largo plazo
```

**Los repasos son cortos a propósito.** No se relee la nota completa: se intenta **recordar primero** y solo se abre la nota para verificar. Releer sin intentar recordar da sensación de saber sin retención real — es el autoengaño más común al estudiar.

## Dónde entra en el horario, sin agregar tiempo

En el bloque 💼 (la hora fragmentada de la jornada), que ya es pasivo y ya tolera interrupciones. No hace falta sumar horas:

| Semana | Bloque 💼 (60 min) |
|---|---|
| Semana 1 | Lectura del día, como estaba — todavía no hay nada que repasar |
| Semanas 2 y 3 | **30 min repaso espaciado + 30 min lectura del día** |

Es el único cambio al horario. El total sigue siendo 3h45.

## Calendario concreto de repasos

Cada día, en el bloque 💼, repasar lo que indica la columna derecha. Sin releer todo: **cerrar la nota, intentar explicarlo, y solo después verificar**.

### Semana 2 — repasa la semana 1

| Día | Tema nuevo del día | 💼 Repasar (5-10 min c/u) | ☐ |
|---|---|---|---|
| Lun 7 sep | Spring core, Lombok | Capas y hexagonal *(+7)* | ☐ |
| Mar 8 sep | JPA, transacciones | SAGA y Cache-Aside *(+7)* | ☐ |
| Mié 9 sep | .NET async, EF Core | Singleton, Factory, Circuit Breaker *(+7)* | ☐ |
| Jue 10 sep | Índices, NoSQL | Repository y Strategy *(+7)* · Spring/Lombok *(+3)* | ☐ |
| Vie 11 sep | Simulacro | JPA y transacciones *(+3)* | ☐ |

### Semana 3 — repasa las semanas 1 y 2

| Día | Tema nuevo del día | 💼 Repasar (5-10 min c/u) | ☐ |
|---|---|---|---|
| Lun 14 sep | OAuth2, OIDC, OWASP | Spring/Lombok *(+7)* · .NET async/EF *(+5)* | ☐ |
| Mar 15 sep | Kafka, idempotencia, Outbox | JPA y transacciones *(+7)* · **Capas y hexagonal** *(+14)* | ☐ |
| Mié 16 sep | Microservicios, observabilidad | Índices y NoSQL *(+6)* · **SAGA y Cache-Aside** *(+14)* | ☐ |
| Jue 17 sep | Docker, CI/CD, Kubernetes | Seguridad OAuth *(+3)* · **Patrones creacionales/CB** *(+14)* | ☐ |
| Vie 18 sep | Simulacro completo | Mensajería *(+3)* · **Repository y Strategy** *(+14)* | ☐ |

### Después del 20 de septiembre — el repaso de +21

Los temas de la semana 1 cumplen sus 21 días entre el 21 y el 25 de septiembre. Ese es el repaso que fija a largo plazo, y el que casi nadie hace:

| Fecha | Repasar (tema del +21) | ☐ |
|---|---|---|
| Lun 21 sep | Capas y arquitectura hexagonal | ☐ |
| Mar 22 sep | SAGA y Cache-Aside | ☐ |
| Mié 23 sep | Singleton, Factory, Circuit Breaker | ☐ |
| Jue 24 sep | Repository y Strategy | ☐ |
| Vie 25 sep | Historias conductuales | ☐ |
| Lun 28 sep | Spring core y Lombok | ☐ |
| Mar 29 sep | JPA y transacciones | ☐ |
| Mié 30 sep | .NET async y EF Core | ☐ |
| Jue 1 oct | Índices, ACID/BASE, NoSQL | ☐ |
| Vie 2 oct | Seguridad y OWASP | ☐ |

A partir de octubre el plan continúa con las semanas 4-8 de [[07 - Plan de ocho semanas]], y cada tema nuevo entra en la misma cadena de +3/+7/+21.

## Cómo hacer un repaso de 5 minutos bien

El orden importa. Invertirlo lo convierte en lectura pasiva:

1. **Cerrar la nota.** Sin mirar.
2. **Decir en voz alta** lo que recordás del tema: definición, ejemplo, trade-off.
3. **Abrir la nota** y comparar contra el checklist del final.
4. **Anotar solo lo que faltó** — eso es lo único que se vuelve a repasar.

Si en el paso 2 no sale nada, no es un fracaso: es la señal de que ese tema necesitaba este repaso. Anotarlo y seguir.

## Qué priorizar si no alcanza el tiempo

Cuando un día no da para todos los repasos indicados:

1. Primero el **+3** — es el más frágil; si se pierde, el tema se borra casi entero.
2. Después el **+21** — es el que convierte "lo estudié" en "lo sé".
3. El **+7** es el más sacrificable de los tres.

Y siempre antes que cualquier repaso: si hay una entrevista agendada, **el repaso de esa vacante manda** sobre el calendario.

## Cómo se conecta con el banco de preguntas

[[20 - Banco de Preguntas de las 14 Vacantes]] ya es un sistema de repaso encubierto: las preguntas marcadas 🟡 y ❌ vuelven a la rotación. La diferencia:

| | Qué repasa |
|---|---|
| Banco de preguntas | Lo que **fallaste** — corrige debilidades |
| Repaso espaciado | Lo que **acertaste** — evita perder lo ganado |

Los dos hacen falta. Repasar solo los errores deja que lo aprendido se caiga en silencio, que es exactamente lo que pasa cuando alguien dice "esto lo sabía hace un mes".
