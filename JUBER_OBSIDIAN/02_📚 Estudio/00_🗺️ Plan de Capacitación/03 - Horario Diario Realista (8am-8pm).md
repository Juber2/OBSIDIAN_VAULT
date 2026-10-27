---
tags:
  - Postulaciones
  - Horario
  - Rutina
Creado: 2026-08-31
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - 07 - Plan de ocho semanas
  - Guía Maestra - Mi Recomendación de Preparación
---

# Horario diario realista (jornada 8am-8pm)

[[07 - Plan de ocho semanas]] define bloques de "90 minutos / 2 horas / 4 horas" sin atarlos al reloj. Con una jornada de **8am a 8pm** (12 horas), la rutina "Intensiva — 4 horas" de esa nota no entra ningún día de semana. Esta nota reemplaza esa ambigüedad por horarios de reloj concretos, calibrados a lo que realmente queda libre.

> [!important]
> Hoy, **31 de agosto**, es el día 1 según [[00 - Empezar a postular mañana]]. Este horario arranca hoy mismo, no la próxima semana.

> [!tip] El temario día a día está en la nota 19
> Esta nota define **la estructura horaria** (cuándo y de qué tipo). [[04 - Plan Detallado 3 Semanas (dia a dia)]] la llena con **qué estudiar cada día concreto** del 31 ago al 20 sep, con checkboxes por concepto. Usá esta para entender el porqué del horario, y la 19 para ejecutar.

## El cálculo honesto

```
06:00 ────────── 08:00   →  2h libres antes del trabajo
08:00 ────────── 20:00   →  12h de jornada, con ~1h aprovechable en fragmentos
20:00 ────────── 22:15   →  ~2h libres después del trabajo (descontando cena/corte)
22:15 ────────── 06:00   →  dormir — innegociable
```

**Total entre semana: ~3h45 por día.**

| Bloque | Duración | Tipo |
|---|---:|---|
| Postulaciones (mañana) | 30 min | Ejecución |
| Estudio activo (mañana) | 45 min | **Activo** |
| Durante la jornada | 60 min | **Pasivo/repaso**, fragmentado |
| Bloque principal (noche) | 75 min | **Activo** |
| Registro | 15 min | Cierre |
| **Total** | **3h45** | |

> [!warning] Las tres horas de estudio no son intercambiables
> 45 min de mañana + 75 min de noche son **estudio activo**: hacer el ejercicio, escribir código, responder en voz alta. La hora de la jornada es **pasiva**, porque va interrumpida — sirve para leer y repasar, no para aprender algo nuevo. Tratarlas como equivalentes es el error que hace que "estudié 3 horas" no se traduzca en poder responder en una entrevista.

## Lunes a viernes

| Horario | Bloque | Qué hacer | Por qué a esa hora |
|---|---|---|---|
| 06:00–06:15 | Arranque | Despertar, café, sin pantalla de trabajo todavía | Evitar arrancar el día ya mirando el chat del cliente |
| 06:15–06:45 | **Postulaciones** (30 min) | Revisar 1-2 vacantes nuevas, enviar una postulación adaptada, o un seguimiento — trabajo mecánico, no de aprendizaje | Recién despierto rinde para tareas de ejecución, no para retener conceptos nuevos |
| 06:45–07:30 | **Estudio activo** (45 min) | La "prueba" del día del tema que toque (ver rotación abajo) — nunca lectura pasiva | Es el único momento del día con la cabeza fresca y sin interrupciones del cliente |
| 07:30–08:00 | Cierre | Prepararse para la jornada laboral | — |
| **08:00–20:00** | **Trabajo + 1h de repaso fragmentado** | Repartida en 2-3 tramos, **no** un bloque seguido: ~20 min media mañana, ~20 min post-almuerzo, ~20 min media tarde. Solo material pasivo (ver abajo) | En una jornada de 12h siempre hay tiempo muerto real: builds, deploys, esperar respuesta del cliente, la caída de energía post-almuerzo |
| 20:00–20:45 | Corte | Cena, desconectar del todo | El cerebro necesita el corte entre "modo cliente" y "modo estudio", si no, rinde peor en ambos |
| 20:45–22:00 | **Bloque principal** (75 min) | Un solo tema, nunca tres — ver rotación | Es el bloque más largo del día; mezclarlo dispersa en vez de fijar |
| 22:00–22:15 | Registro | 5 minutos: qué avancé, qué toca mañana ([[00 - Empezar a postular mañana]] tiene la tabla) | Cierra el día sin cargar pendientes mentales a la cama |
| 22:15 | Dormir | Sin pantallas | Con jornada de 12h, dormir mal degrada el trabajo *y* el estudio al día siguiente |

## Qué sí y qué no entra en la hora de la jornada

El criterio es uno solo: **¿sobrevive a que te interrumpan a los 4 minutos?** Si perder el hilo te obliga a empezar de nuevo, ese material no va acá — va al bloque de la noche.

| ✅ Sirve fragmentado | ❌ No sirve fragmentado |
|---|---|
| Leer una nota del vault (Azure, patrones, Spring) | Codear el [[06 - Laboratorio Integrado - API de Ordenes]] |
| Repasar el [[15 - Banco de preguntas Full Stack]] — 3-4 preguntas por tramo | Aprender un concepto nuevo y difícil desde cero |
| Leer documentación oficial de algo puntual | Montar un entorno o depurar un error propio |
| Revisar la tabla de equivalencias .NET ↔ Java de [[06 - Ruta Java Spring Boot como segundo eje backend]] | Simulacro de entrevista en voz alta |
| Escanear una vacante nueva y anotarla | Escribir tests o refactorizar |
| Releer un "Errores comunes" o "Checklist" de cualquier nota | Cualquier cosa que necesite estado de flujo |

### La versión de máximo rendimiento: aprovechar el solapamiento

La hora más productiva del día no es la que estudia algo desconectado del trabajo, sino la que **estudia lo que estás por aplicar en el proyecto del cliente ese mismo día**. Si esta semana tocás autenticación en StaffTrainingPlatform, que la hora fragmentada sea [[04 - Entra ID y Seguridad de APIs]]; si tocás performance de queries, que sea EF Core y planes de ejecución.

Ese solapamiento tiene dos ventajas que ningún otro bloque da:

1. **Retenés mucho más**, porque lo aplicás en horas, no en semanas.
2. **Genera evidencia real** para el CV y la entrevista — "lo hice en un proyecto", que es la única evidencia que [[01 - Diagnóstico y matriz de competencias]] considera válida. Estudiar en abstracto no produce eso.

Cuando no haya solapamiento posible, cae en el material pasivo de la tabla de arriba, alineado al tema del día según la rotación.

## Rotación semanal del bloque de 75 minutos

No estudiar "lo que corresponda" cada noche — fijar un tema por día evita el efecto de abrir tres notas y no cerrar ninguna.

| Día | Foco | Material |
|---|---|---|
| Lunes | .NET | [[04 - Ruta de repaso NET para entrevistas]] |
| Martes | Java/Spring | [[06 - Ruta Java Spring Boot como segundo eje backend]] |
| Miércoles | Azure práctico | Laboratorios 01-06 en `01_☁️ Azure/03_🧪 Laboratorios/` |
| Jueves | Arquitectura y patrones | `ARQUITECTURA.md` + bloque de arquitectura de [[15 - Banco de preguntas Full Stack]] |
| Viernes | Cierre liviano | Postulaciones intensivo + preguntas conductuales — **no** contenido técnico nuevo un viernes a la noche |

Viernes es deliberadamente el día más liviano de la semana: es donde más se abandona un plan de estudio por cansancio acumulado, así que se reserva para repaso y postulación, no para aprender algo nuevo.

## Fin de semana

| Bloque | Duración | Qué hacer |
|---|---|---|
| Sábado mañana | 3h | Bloque grande de proyecto — [[06 - Laboratorio Integrado - API de Ordenes]] avanza acá, no entre semana. Necesita concentración larga que 75 minutos no dan. |
| Sábado tarde/noche | Libre | Descanso real. Sin culpa — es lo que sostiene las 8 semanas. |
| Domingo | 1h | Revisión de [[07 - Plan de ocho semanas]] (los 5 puntos de domingo) + un simulacro corto de entrevista + ajustar la semana siguiente |

## Total semanal

```
Entre semana:  (30 + 45 + 60 + 75) min × 5 días   = 17.5 h
Fin de semana: 3h (sábado) + 1h (domingo)          =  4 h
                                                    ─────────
Total                                               ≈ 21.5 h/semana
```

Desglosado por tipo, que es lo que importa:

| Tipo | Horas/semana |
|---|---:|
| Estudio **activo** (mañana + noche + sábado) | 13 h |
| Repaso **pasivo** (hora de la jornada) | 5 h |
| Postulaciones y seguimiento | 2.5 h |
| Revisión de domingo | 1 h |

Las 13h de estudio activo son las que mueven la aguja. Las 5h pasivas multiplican esas 13 —refuerzan lo ya visto— pero **no las reemplazan**: una semana con 5h pasivas y 0 activas no avanza nada, aunque el total diga 5 horas.

## Señal de que la hora de la jornada no está funcionando

Es el bloque más fácil de autoengañarse, porque "estuve leyendo" cuenta como estudiar. Revisá el domingo:

- ¿Podés explicar en voz alta algo que leíste esta semana en esa hora? Si no, era lectura pasiva sin retención — pasá ese material al bloque de la noche.
- ¿Se convirtió en scroll de vacantes o documentación al azar? Volvé a atarla a la rotación del día.
- ¿Está afectando tu trabajo del cliente? Entonces reducila a 30 min. El freelance es lo que paga hoy, y una mala entrega cuesta más que una semana de estudio.

## Qué hacer si un día se cae el plan

Va a pasar — un cliente con una entrega urgente, una mala noche. La regla:

1. **No intentar "recuperar" duplicando el bloque siguiente.** 150 minutos un día no rinden el doble; generan abandono al día siguiente por agotamiento.
2. Si solo queda tiempo para uno de los tres bloques (postulaciones / estudio mañana / estudio noche), priorizar **postulaciones** — es lo único con fecha límite real (la vacante puede cerrarse).
3. Un día perdido no mueve la fecha de cierre de [[07 - Plan de ocho semanas]] — la revisión de domingo es donde se reajusta, no cada noche.

## Ajuste si cambia la disponibilidad

Este horario asume 8am-8pm fijo. Si en algún tramo del freelance hay menos carga (entre proyectos, por ejemplo), el bloque de la mañana puede extenderse a 07:00 y sumar una hora — pero el default siempre vuelve a este horario, no al revés.
