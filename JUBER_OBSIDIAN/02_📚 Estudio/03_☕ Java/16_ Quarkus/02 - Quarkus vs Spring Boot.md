---
tags:
  - Java
  - Quarkus
Creado: 2026-08-09
Relacionado:
  - Quarkus
  - Spring Boot
---

# Quarkus vs Spring Boot

No es "cuál es mejor" en abstracto — es una decisión de contexto. Comparación honesta:

| Criterio | Spring Boot | Quarkus |
|---|---|---|
| Arranque (JVM mode) | 2-5s típico | ~0.5-1s típico |
| Arranque (native) | no aplica de forma nativa/madura | 10-50ms |
| Memoria en runtime | mayor (proxies, reflection en runtime) | menor, especialmente en native |
| Madurez del ecosistema | muy alta, décadas de librerías/integraciones | alta pero más joven, cobertura creciente |
| Curva de aprendizaje si ya sabés Spring | — | baja-media (conceptos similares, distinta implementación) |
| Comunidad y mercado laboral | dominante | creciendo, fuerte en shops Red Hat/OpenShift |
| Compilación nativa | soporte parcial (Spring Native/AOT desde Spring Boot 3) | ciudadano de primera clase desde el diseño |
| Live reload en dev | DevTools (restart, no siempre instantáneo) | hot reload real, más rápido |

## Cuándo elegir cada uno

**Elegí Quarkus si:**
- El servicio corre en Kubernetes con autoscaling agresivo (muchas instancias chicas escalando rápido).
- Es una función serverless o algo con cold-start crítico.
- Memoria es un costo real (muchas réplicas × footprint chico = ahorro directo).
- El equipo puede invertir tiempo en aprender un ecosistema algo distinto.

**Elegí Spring Boot si:**
- Necesitás una librería/integración específica que solo tiene soporte maduro en Spring.
- El equipo ya tiene expertise profundo en Spring y el costo de cambio no se justifica.
- La app es un monolito o servicio de vida larga sin necesidad de arranque ultrarrápido.
- El ecosistema de terceros (herramientas internas, tooling corporativo) ya asume Spring.

## Nota sobre Spring Boot 3+ y Native

Desde Spring Boot 3 (con Spring Native / GraalVM support vía Spring AOT), Spring también puede compilar a nativo — la brecha se achicó, pero Quarkus sigue teniendo ventaja porque el modelo build-time fue el diseño original, no un agregado posterior. Esto es relevante: la comparación no es estática, ambos ecosistemas evolucionan.

## En la práctica

Para un dev que ya domina Spring, aprender Quarkus no es empezar de cero: los conceptos (DI, REST, ORM, config por profiles) son análogos, cambia la implementación y algunas convenciones. La inversión de aprendizaje es razonable si el contexto (K8s-heavy, microservicios chicos) lo justifica.

## Checklist
- [ ] Puedo argumentar con datos (no moda) cuándo usar Quarkus vs Spring Boot.
- [ ] Entiendo que Spring Boot 3+ también soporta compilación nativa, y en qué se diferencia del approach de Quarkus.
- [ ] Sé identificar si un proyecto real se beneficiaría de migrar o no.
