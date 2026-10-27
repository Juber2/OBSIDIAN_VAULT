---
aliases:
  - NeedDoer
tipo: proyecto
estado: definición funcional
creado: 2026-10-17
actualizado: 2026-10-17
tags:
  - proyecto
  - app
  - marketplace
  - servicios-tecnicos
---

# NeedDoer

> Centro de documentación del proyecto. Esta carpeta es la fuente de trabajo para revisar, decidir y actualizar sus funcionalidades.

## Estado actual

- **Fase:** definición funcional.
- **Propuesta:** conectar clientes que necesitan una reparación en casa con técnicos verificados que compiten mediante cotizaciones.
- **Modelo inicial:** el cliente publica solicitudes gratis. El prestador necesita créditos disponibles para cotizar, pero el crédito solo se consume cuando el cliente acepta su cotización.
- **Plataformas:** una app móvil para cliente y prestador, más un panel web de administración.
- **Mercado inicial:** gasfitería, electricidad, electrodomésticos y cerrajería.

## Documentación

1. [[01 - Visión y alcance]] — propósito, actores, flujo principal y límites.
2. [[02 - Mapa de funcionalidades]] — inventario completo de funcionalidades y prioridades; es la referencia principal que iremos editando.
3. [[03 - Reglas de negocio]] — parámetros que afectan varias pantallas y procesos.
4. [[04 - Decisiones pendientes]] — puntos abiertos que requieren una decisión explícita.
5. [[05 - Roadmap y backlog]] — organización por etapas para convertir el mapa en trabajo ejecutable.
6. [[06 - Registro de cambios]] — historial de cambios importantes en el alcance.
7. [[07 - Prototipo funcional]] — alcance, arquitectura local y criterios del prototipo.

## Criterio de prioridad

| Prioridad | Significado |
| --- | --- |
| MVP | Necesario para lanzar la primera versión. |
| Fase 2 | Candidato para la siguiente versión después de validar el MVP. |
| Por decidir | Aún requiere análisis o una decisión de producto. |

## Enfoque del prototipo

El trabajo actual será un prototipo funcional de extremo a extremo para probar la idea, los recorridos, las reglas y la interacción entre los tres actores. Se realizará con React y herramientas locales sencillas; esta elección no define todavía la tecnología del producto final.

El prototipo tendrá:

- Una experiencia móvil simulada para cliente y prestador.
- Un panel web sencillo para administración.
- Una API local y una base de datos SQLite compartida.
- Datos de prueba que permitan recorrer casos realistas.
- Los recorridos necesarios para demostrar y validar el producto funcionando.

El prototipo tendrá lógica real dentro del alcance elegido. Los servicios externos como SMS, pagos, mapas, notificaciones push y validación de identidad se simularán localmente. No requiere publicación en tiendas ni infraestructura de producción.

## Próxima revisión

- [ ] Validar el recorrido completo de una solicitud y sus estados.
- [ ] Revisar si cada función marcada como MVP es indispensable para el primer lanzamiento.
- [ ] Resolver las decisiones críticas de [[04 - Decisiones pendientes]].
- [ ] Convertir el MVP aprobado en historias de usuario y criterios de aceptación.
- [ ] Usar el prototipo para validar los recorridos antes de definir arquitectura, integraciones, costos y calendario de desarrollo.

## Principio de edición

Cuando cambie una funcionalidad, se actualiza primero [[02 - Mapa de funcionalidades]]. Si el cambio afecta una regla, una decisión o una etapa, también se actualiza la nota correspondiente y se registra en [[06 - Registro de cambios]].
