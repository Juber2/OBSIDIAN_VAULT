---
tags:
  - Linux
  - DevOps
  - Linux-Senior
Creado: 2026-07-28
Relacionado:
  - Senior Linux DevOps
  - LINUX
---

# 05 - Decisiones de Arquitectura de Infraestructura

## Que vas a aprender

En este capitulo vas a cerrar el libro consolidando el criterio senior para tomar **decisiones de arquitectura de infraestructura**: el mismo marco de decisiones ya visto en los libros de Docker y NestJS, ahora aplicado especificamente a eleccion de infraestructura Linux completa.

## Explicacion

Las decisiones de infraestructura (VPS unico versus cluster, distribucion, nivel de automatizacion, estrategia de HA) siguen el mismo principio de todo este ecosistema de libros: no se trata de aplicar "la mejor practica" en abstracto, sino de evaluar que problema real resuelve cada decision para el contexto especifico del equipo y del negocio. Un servidor unico bien administrado (con backups solidos, monitoreo, y actualizaciones de seguridad automatizadas) puede ser completamente apropiado para un proyecto pequeño, mientras que un sistema de alta disponibilidad completo con clustering seria sobre-ingenieria costosa y dificil de mantener para ese mismo contexto.

El marco de decision es el mismo aplicado consistentemente en toda esta serie de libros: que problema real resuelve, que pasa si no se hace, que complejidad introduce, es reversible, y el equipo tiene la experiencia para mantenerlo bien. Esto conecta explicitamente con el capitulo de "cuando migrar de Compose a un orquestador" del libro de Docker: la misma logica de decision aplica a nivel de infraestructura Linux completa, no solo a nivel de containers.

## Modelo mental

Pensa cada decision de infraestructura como elegir el vehiculo correcto para un viaje especifico, no el vehiculo "objetivamente mejor" en abstracto: un auto compacto es la eleccion correcta para trayectos urbanos cotidianos, aunque un camion tenga mas capacidad; el camion se justifica solo cuando el trayecto especifico realmente requiere esa capacidad, no por default.

## Ejemplo base

```
Marco de decision para infraestructura Linux (aplicando el mismo criterio de otros libros):

1. Que problema real resuelve esta decision de infraestructura, hoy, en este contexto?
2. Que pasa si NO se hace (costo real de no invertir en esto)?
3. Que complejidad operativa adicional introduce (aprendizaje, mantenimiento)?
4. Es reversible? Que tan caro seria deshacer esta decision en 6 meses?
5. El equipo tiene la experiencia para operar esto con confianza, o se estaria improvisando?
```

## Como se usa en un proyecto real

1. Se aplica este marco antes de decisiones de infraestructura significativas: migrar de un servidor unico a clustering, adoptar IaC completo, invertir en alta disponibilidad.
2. Se empieza con la infraestructura mas simple que resuelve el problema real actual, escalando en complejidad solo cuando el dolor real de no tenerlo se vuelve evidente, el mismo principio ya visto para arquitectura de aplicacion en NestJS y orquestacion en Docker.
3. Se documenta cada decision significativa como un ADR, incluyendo las alternativas descartadas y por que, siguiendo el mismo patron de documentacion tecnica de toda esta serie de libros.
4. Se revisa periodicamente si las decisiones de infraestructura tomadas siguen siendo apropiadas a medida que el proyecto y el equipo crecen.
5. Se resiste la presion de adoptar patrones de infraestructura complejos "porque es lo que se usa en la industria" sin justificacion real para el contexto especifico.

## Criterio junior

Un junior debe poder seguir decisiones de infraestructura ya tomadas, entendiendo el razonamiento si esta bien documentado.

## Criterio mid

Un mid debe aplicar el marco de decision a un caso concreto de su experiencia, cuestionando constructivamente decisiones que no encajan con la realidad del proyecto.

## Criterio senior

Un senior debe liderar decisiones de arquitectura de infraestructura para toda la organizacion, aplicando el mismo rigor de evaluacion visto en las decisiones de arquitectura de aplicacion (NestJS) y de orquestacion de containers (Docker), documentando y revisando esas decisiones con el tiempo.

## Errores comunes

- Adoptar infraestructura compleja (clustering, alta disponibilidad completa) por moda o presion externa, sin que el problema real del proyecto lo justifique.
- No documentar el razonamiento detras de decisiones de infraestructura significativas, dejando que el equipo futuro repita debates ya resueltos.
- No revisar decisiones de infraestructura periodicamente, quedando atado a una eleccion que ya no encaja con la realidad actual del proyecto.
- Aplicar un criterio distinto para decisiones de infraestructura que el ya usado consistentemente para decisiones de arquitectura de aplicacion y de containers.

## Practica

- [ ] Aplica el marco de cinco preguntas a una decision de infraestructura real o hipotetica de tu contexto.
- [ ] Documenta esa decision como un ADR, incluyendo alternativas descartadas.
- [ ] Identifica una decision de infraestructura de un proyecto que conozcas que ya no encaja con su realidad actual.
- [ ] Compara este marco de decision con el ya visto para orquestacion de containers en el libro de Docker.
- [ ] Reflexiona sobre todo el libro de Linux: que capitulos aplicarias primero en un servidor nuevo.

## Preguntas de repaso

- Por que la pregunta correcta para una decision de infraestructura nunca es "es esto una buena practica" en abstracto?
- Como se relaciona este marco de decision con el ya visto para migrar de Compose a un orquestador en el libro de Docker?
- Por que empezar con la infraestructura mas simple que resuelve el problema real es generalmente la decision correcta?
- Como te asegurarias de que una decision de infraestructura se revise cuando el contexto del proyecto cambie?

## Cierre del libro

Completar este capitulo cierra el recorrido completo de Linux junior a senior: desde los fundamentos del kernel y distribuciones, pasando por dominio completo de la terminal y scripting, redes, gestion de paquetes, almacenamiento, seguridad, administracion de servidores y troubleshooting, hasta el criterio senior de infraestructura como codigo, alta disponibilidad, y decisiones de arquitectura. Este conocimiento es la base tecnica que sostiene todo lo aprendido en los libros de [[NESTJS|NestJS]] y [[DOCKER|Docker]]: cada container, cada red, cada volumen, en el fondo corre sobre exactamente estos mismos fundamentos de Linux.
