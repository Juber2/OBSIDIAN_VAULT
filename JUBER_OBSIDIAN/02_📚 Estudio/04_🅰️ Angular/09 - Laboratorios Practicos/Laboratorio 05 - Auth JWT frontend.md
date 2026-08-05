---
tags:
  - angular
  - laboratorio
  - seguridad
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos Angular
  - ANGULAR
---

# Laboratorio 05 - Auth JWT frontend

## Objetivo

Implementar la parte frontend de autenticación: login, storage elegido, interceptor, guard, refresh flow y logout seguro.

## Piezas

- `AuthStore`.
- `authInterceptor`.
- `authGuard`.
- Manejo de 401 con `TOKEN_EXPIRED`.
- Limpieza de sesión si refresh falla.

## Flujo

El cliente no valida permisos reales. Solo mejora UX. El backend decide autorización. Angular adjunta token, reacciona a 401 y protege rutas visualmente.

## Interceptor mental model

Evita que el interceptor tenga demasiadas responsabilidades. Auth agrega token; error interceptor maneja errores globales; loading interceptor si aplica maneja loading.

## Errores comunes

- Creer que guard es seguridad real
- No diferenciar 401 y 403
- Refresh infinito por interceptor mal diseñado

## Practica

- [ ] Crear login
- [ ] Crear guard
- [ ] Simular TOKEN_EXPIRED y refresh

## Relacionado

- [[Laboratorios Practicos Angular]]
- [[ANGULAR]]

## Desarrollo guiado

Este laboratorio no se debe resolver copiando snippets aislados. La idea es construir una pequeña pieza funcional, ejecutar la app y registrar qué decisiones tomaste. Para **Laboratorio 05 - Auth JWT frontend**, trabaja en tres capas: primero haz que funcione, luego ordena responsabilidades y finalmente agrega validación o prueba mínima.

## Pasos sugeridos

1. Crea una rama o carpeta de práctica con nombre claro.
2. Implementa el flujo mínimo sin optimizar antes de tiempo.
3. Separa UI, estado y acceso a datos si el ejercicio lo justifica.
4. Agrega al menos un caso de error: API falla, formulario inválido, sesión expirada o lista vacía.
5. Escribe una nota corta con lo que aprendiste y lo que cambiarías.

## Criterio de revisión

El laboratorio está completo cuando otra persona puede abrir el código, entender el flujo y ejecutarlo. Revisa nombres, estados de UI, manejo de errores y que no hayas escondido reglas importantes dentro del template.
