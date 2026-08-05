---
tags:
  - NestJS
  - TypeScript
  - Nest-Senior
Creado: 2026-07-27
Relacionado:
  - Senior NestJS
  - NESTJS
---

# 02 - Decisiones Arquitectonicas

## Que vas a aprender

En este capitulo vas a desarrollar el criterio para tomar y comunicar **decisiones arquitectonicas** en un proyecto NestJS: como evaluar tradeoffs, cuando adoptar un patron mas complejo (visto en el capitulo de Arquitectura) y cuando resistir esa tentacion.

## Explicacion

La habilidad senior mas dificil de enseñar no es conocer los patrones (DDD, CQRS, hexagonal, microservicios), sino saber **cuando no usarlos**. Cada patron resuelve un problema especifico a costa de complejidad adicional; aplicarlo donde el problema no existe todavia es pagar el costo sin el beneficio. La pregunta correcta nunca es "es esto una buena practica" en abstracto, sino "este problema especifico, en este proyecto especifico, con este equipo especifico, justifica esta complejidad".

Una decision arquitectonica seria siempre considera: la complejidad real del dominio, el tamaño y experiencia del equipo, el tiempo disponible, y la reversibilidad de la decision (algunas decisiones son faciles de revertir despues, otras no).

## Modelo mental

Pensa cada patron arquitectonico como una herramienta especializada, no como una "mejor practica universal": un martillo neumatico es una excelente herramienta para romper concreto, pero es una mala eleccion para colgar un cuadro. El trabajo senior es reconocer que herramienta corresponde a que problema real, no aplicar la mas sofisticada disponible por default.

## Ejemplo base

```
Marco simple para evaluar una decision arquitectonica:

1. Que problema real resuelve esto, hoy, en este proyecto?
2. Que pasa si NO lo hacemos (cual es el costo de no adoptarlo)?
3. Que complejidad extra introduce (para el equipo, para el onboarding, para el mantenimiento)?
4. Es reversible? Que tan caro seria deshacer esta decision en 6 meses?
5. El equipo tiene la experiencia para mantener esto bien, o vamos a improvisar sobre la marcha?
```

## Como se usa en un proyecto real

1. Toda decision arquitectonica significativa se documenta con un ADR (ver capitulo de Ecosistema y Calidad), incluyendo las alternativas descartadas y por que.
2. Se prefieren decisiones reversibles sobre irreversibles cuando hay incertidumbre real sobre el camino correcto.
3. Se empieza simple (monolito modular, capas basicas) y se migra a patrones mas complejos solo cuando el dolor real de no tenerlos se vuelve evidente, no de forma anticipada.
4. Se involucra al equipo en decisiones grandes, no solo se imponen desde arriba, para asegurar comprension y compromiso real con la decision.
5. Se revisan decisiones pasadas periodicamente: una decision correcta para un equipo de 3 personas puede no serlo para uno de 20.

## Criterio junior

Un junior debe poder seguir una decision arquitectonica ya tomada, entendiendo el razonamiento detras si esta bien documentada.

## Criterio mid

Un mid debe poder cuestionar constructivamente una decision arquitectonica cuando ve una señal de que no encaja con la realidad del proyecto, aportando datos concretos.

## Criterio senior

Un senior debe liderar el proceso de toma de decisiones arquitectonicas del equipo, resistir la presion de adoptar patrones de moda sin justificacion real, y ser responsable de que esas decisiones se revisen y ajusten a medida que el contexto cambia.

## Errores comunes

- Adoptar un patron complejo (microservicios, CQRS, DDD completo) por moda o por curriculum, sin que el problema real lo justifique.
- No documentar el razonamiento detras de una decision, dejando que el equipo futuro repita debates ya resueltos o mantenga algo sin entender por que existe.
- Tomar decisiones irreversibles con poca informacion cuando una alternativa reversible hubiera permitido aprender mas antes de comprometerse.
- Nunca revisar decisiones pasadas, manteniendo arquitectura que ya no encaja con la realidad actual del equipo o del negocio.

## Practica

- [ ] Aplica el marco de cinco preguntas a una decision arquitectonica real o hipotetica de tu proyecto.
- [ ] Escribe un ADR completo para esa decision, incluyendo alternativas descartadas.
- [ ] Identifica una decision arquitectonica de un proyecto que conozcas que ya no encaja con la realidad actual del equipo.
- [ ] Practica argumentar por que NO adoptarias un patron de moda en un caso especifico.
- [ ] Anota un ejemplo de decision reversible vs irreversible en tu contexto actual.

## Preguntas de repaso

- Por que la pregunta correcta nunca es "es esto una buena practica" en abstracto?
- Que factores deberia considerar una decision arquitectonica seria, mas alla de lo tecnico?
- Por que preferir decisiones reversibles cuando hay incertidumbre?
- Como te asegurarias de que una decision arquitectonica se revise cuando el contexto del equipo cambie?
