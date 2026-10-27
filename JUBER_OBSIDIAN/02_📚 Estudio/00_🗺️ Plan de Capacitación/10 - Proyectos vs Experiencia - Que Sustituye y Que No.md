---
tags:
  - Estudio
  - Carrera
  - Evidencia
Creado: 2026-08-31
Relacionado:
  - PLAN DE CAPACITACION
  - 09 - Perfil Objetivo - Puestos con Peso
---

# Proyectos vs experiencia — qué sustituye y qué no

La pregunta de fondo del plan: **¿un proyecto propio reemplaza experiencia laboral?** La respuesta honesta es *depende de qué*, porque "experiencia" son cuatro cosas distintas y los proyectos cubren dos.

## Las cuatro cosas que se llaman experiencia

| Qué evalúan en realidad | ¿Un proyecto lo sustituye? | Por qué |
|---|---|---|
| **Manejo de la tecnología** — ¿sabe Spring? ¿sabe AWS? | 🟢 **Sí, muy bien** | Un repo público con código tuyo responde esto igual o mejor que un empleo: pueden **verlo** |
| **Haber peleado con producción** — incidentes, datos raros, cosas que fallan de noche | 🟡 **Parcial** | Romper el laboratorio a propósito te acerca. No es lo mismo que un incidente real con usuarios afectados, y el entrevistador lo nota |
| **Años (el número)** | 🔴 **No** | Si el filtro dice "5 años", ningún proyecto los crea. Muchas veces es un filtro automático antes de que un humano lea |
| **Trabajar con restricciones reales** — equipo, deadlines, legacy, negocio, código ajeno | 🔴 **No** | Es lo que "senior" significa de verdad, y en un proyecto propio no existe: no hay nadie que te discuta una decisión |

## El punto que cambia todo tu caso

**Vos no estás reemplazando experiencia: ya tenés 5 años de backend real.**

No sos alguien saliendo de un bootcamp intentando aparentar. Tenés cuatro empresas, sistemas en producción, integraciones con terceros, migraciones legacy, optimización de queries y un pipeline de CI/CD que armaste vos.

Lo que te falta no es *experiencia*. Es **una tecnología puntual dentro de una experiencia que ya existe**. Eso es una brecha completamente distinta y mucho más fácil de cubrir.

```
Lo que tenés          →  5 años de backend, producción, equipo, dominio
Lo que falta          →  Java / AWS como tecnología
Lo que lo cubre       →  Un proyecto propio, desplegado y defendible
```

La frase honesta que resulta de eso:

> *"Tengo cinco años construyendo backend empresarial: APIs, integraciones, bases de datos, despliegue. Esa experiencia es en .NET, porque es el stack de las empresas donde estuve. Java lo trabajé en proyectos propios — acá está el repo, te muestro cómo resolví la idempotencia."*

Eso se sostiene ante cualquier repregunta. **Lo que no se sostiene** es "tengo 3 años de Java" cuando no los tenés.

## Qué hace que un proyecto cuente de verdad

La mayoría no cuentan. Estos seis criterios separan un repo que suma de uno que el entrevistador ignora:

| | Criterio | Por qué |
|---|---|---|
| 1 | **Desplegado y accesible** | Una URL que abre vale más que un repo. Prueba que sabés operar, no solo escribir |
| 2 | **Historial de commits real** | Un solo *"initial commit"* con 4.000 líneas grita tutorial copiado |
| 3 | **README que explica decisiones**, no solo cómo correrlo | Es lo que lo separa de un ejercicio |
| 4 | **Resuelve algo plausible** | Un sistema de órdenes con pagos y facturación, no una lista de tareas |
| 5 | **Tiene las partes difíciles** | Tests, manejo de errores, auth, idempotencia, observabilidad — justo lo que los tutoriales saltean |
| 6 | **Podés defender cada decisión** | El criterio que decide todo. Si no explicás por qué elegiste X, el proyecto juega en contra |

El punto 6 es el filtro real. Un proyecto que no podés defender es peor que no tenerlo: demuestra que copiaste.

## La palanca que casi nadie usa: tu trabajo actual

Acá está lo más importante de esta nota.

Un proyecto propio produce **evidencia de tecnología**. Pero tu empleo actual produce **experiencia real** — y podés dirigir parte de ella hacia donde te falta, sin cambiar de trabajo y sin pedir permiso:

| Si en Pluxee lográs… | Deja de ser "proyecto propio" y pasa a ser **experiencia laboral** |
|---|---|
| Escribir tests para un módulo nuevo | Testing en producción, no en un lab |
| Proponer y documentar una decisión técnica (un ADR) | Diseño de soluciones |
| Introducir idempotencia en un proceso que se reprocesa | Mensajería y resiliencia reales |
| Agregar logging estructurado a un flujo crítico | Observabilidad con usuarios de verdad |
| Meter un análisis estático en el pipeline | DevSecOps |

Cada una de esas líneas vale **más** en el CV que la misma línea en un proyecto personal, y te cuesta menos tiempo que construirla desde cero. El laboratorio cubre lo que tu trabajo no te deja tocar; tu trabajo cubre lo que un laboratorio nunca va a dar.

> [!tip] La combinación es lo que funciona
> Experiencia real (dominio, equipo, producción) **+** proyecto propio (la tecnología puntual). Ninguna de las dos sola alcanza para los puestos con peso.

## Sobre los 4 meses

Tenés razón en que es ajustado. Precisiones:

| Objetivo | ¿4 meses alcanzan? |
|---|---|
| Ser **creíble** trabajando en Java y AWS | 🟢 **Sí** — con un proyecto terminado y defendible |
| Presentarte como **Java senior con años de Spring** | 🔴 No, y tampoco conviene intentarlo |
| Cubrir Cuidafarma o Synopsis | 🟡 Ajustado pero posible |
| Cubrir los puestos de arquitecto | 🔴 No — ahí falta tiempo ejerciendo, no estudio |

Y el error más caro con poco tiempo: **hacer cinco proyectos a medias**. Un proyecto terminado, desplegado y que podés defender línea por línea vale más que cinco repos abandonados en la fase 2 — que además comunican lo contrario de lo que buscás.

Por eso el plan tiene **un** laboratorio y no un portafolio.

## Lo que se puede decir y lo que no

| ✅ Honesto y aceptado | ❌ Se cae en la primera repregunta |
|---|---|
| "Lo implementé en un proyecto propio, te muestro el repo" | "Tengo 2 años de Java" |
| "En producción usé .NET; el equivalente en Java lo trabajé por mi cuenta" | Poner Java en un empleo donde no lo usaste |
| "No me tocó en producción, pero sé cómo lo abordaría y lo probé acá" | Inventar un incidente que no viviste |
| "Estoy formándome en X, ya tengo esto construido" | Listar una tecnología que no podés defender 10 minutos |

La columna izquierda no te cierra puertas: es la respuesta que dan la mayoría de los candidatos que sí entran.

## Checklist del laboratorio como sustituto válido

- [ ] Está **desplegado** y la URL abre
- [ ] El repo es público y tiene historial de commits real
- [ ] El README explica **decisiones**, no solo comandos
- [ ] Tiene tests que corren
- [ ] Tiene las partes difíciles: auth, errores, idempotencia, observabilidad
- [ ] Hay 2-3 ADR con las alternativas que descarté
- [ ] Puedo explicar **cualquier** archivo sin abrirlo primero
- [ ] Puedo contarlo entero en 5 minutos
