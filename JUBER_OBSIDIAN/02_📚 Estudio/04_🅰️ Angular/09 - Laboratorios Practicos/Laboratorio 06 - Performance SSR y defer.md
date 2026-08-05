---
tags:
  - angular
  - laboratorio
  - performance
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos Angular
  - ANGULAR
---

# Laboratorio 06 - Performance SSR y defer

## Objetivo

Medir y optimizar una pantalla pesada usando lazy loading, `@defer`, tracking correcto, budgets y, si aplica, SSR/hydration.

## Escenario

Dashboard con tabla, gráfico pesado y resumen. El gráfico no debe bloquear primer render.

## Uso de defer


```html
@defer (on viewport; prefetch on idle) {
  <app-heavy-chart />
} @placeholder {
  <app-chart-skeleton />
}
```


## SSR criterio

Usa SSR/SSG si necesitas SEO, primera carga pública o contenido indexable. Para un admin privado, probablemente CSR bien optimizado sea suficiente.

## Errores comunes

- Optimizar sin medir
- Diferir contenido crítico
- Agregar SSR por moda

## Practica

- [ ] Medir bundle
- [ ] Aplicar @defer
- [ ] Escribir decisión CSR vs SSR

## Relacionado

- [[Laboratorios Practicos Angular]]
- [[ANGULAR]]

## Desarrollo guiado

Este laboratorio no se debe resolver copiando snippets aislados. La idea es construir una pequeña pieza funcional, ejecutar la app y registrar qué decisiones tomaste. Para **Laboratorio 06 - Performance SSR y defer**, trabaja en tres capas: primero haz que funcione, luego ordena responsabilidades y finalmente agrega validación o prueba mínima.

## Pasos sugeridos

1. Crea una rama o carpeta de práctica con nombre claro.
2. Implementa el flujo mínimo sin optimizar antes de tiempo.
3. Separa UI, estado y acceso a datos si el ejercicio lo justifica.
4. Agrega al menos un caso de error: API falla, formulario inválido, sesión expirada o lista vacía.
5. Escribe una nota corta con lo que aprendiste y lo que cambiarías.

## Criterio de revisión

El laboratorio está completo cuando otra persona puede abrir el código, entender el flujo y ejecutarlo. Revisa nombres, estados de UI, manejo de errores y que no hayas escondido reglas importantes dentro del template.
