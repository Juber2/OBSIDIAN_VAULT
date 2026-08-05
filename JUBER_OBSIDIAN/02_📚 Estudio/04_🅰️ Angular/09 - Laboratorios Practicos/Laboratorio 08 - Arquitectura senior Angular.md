---
tags:
  - angular
  - laboratorio
  - arquitectura
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos Angular
  - ANGULAR
---

# Laboratorio 08 - Arquitectura senior Angular

## Objetivo

Refactorizar la app hacia arquitectura por features con boundaries claros, data-access, UI, facades/adapters si aportan valor y documentación de decisiones.

## Estructura

Cada feature debe exponer lo necesario y ocultar detalles internos. Shared no debe llenarse de dominio. Core no debe convertirse en cajón de sastre.

## Decisiones esperadas

- Signals o RxJS para estado.
- NgRx sí/no.
- CSR/SSR.
- Estrategia de auth frontend.
- Testing strategy.

## Criterio de terminado

Existe README de arquitectura, ADRs cortas y checklist de code review aplicado a la feature.

## Errores comunes

- Carpetas sin boundaries
- Abusar de facades
- NgRx por costumbre

## Practica

- [ ] Reestructurar feature users
- [ ] Escribir ADR
- [ ] Aplicar checklist senior

## Relacionado

- [[Laboratorios Practicos Angular]]
- [[ANGULAR]]

## Desarrollo guiado

Este laboratorio no se debe resolver copiando snippets aislados. La idea es construir una pequeña pieza funcional, ejecutar la app y registrar qué decisiones tomaste. Para **Laboratorio 08 - Arquitectura senior Angular**, trabaja en tres capas: primero haz que funcione, luego ordena responsabilidades y finalmente agrega validación o prueba mínima.

## Pasos sugeridos

1. Crea una rama o carpeta de práctica con nombre claro.
2. Implementa el flujo mínimo sin optimizar antes de tiempo.
3. Separa UI, estado y acceso a datos si el ejercicio lo justifica.
4. Agrega al menos un caso de error: API falla, formulario inválido, sesión expirada o lista vacía.
5. Escribe una nota corta con lo que aprendiste y lo que cambiarías.

## Criterio de revisión

El laboratorio está completo cuando otra persona puede abrir el código, entender el flujo y ejecutarlo. Revisa nombres, estados de UI, manejo de errores y que no hayas escondido reglas importantes dentro del template.
