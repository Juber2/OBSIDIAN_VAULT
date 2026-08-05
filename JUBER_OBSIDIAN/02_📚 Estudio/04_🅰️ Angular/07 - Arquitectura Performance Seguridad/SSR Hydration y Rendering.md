---
tags:
  - Angular
  - Senior
  - Senior
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Arquitectura Performance Seguridad Angular
  - ANGULAR
---

# SSR Hydration y Rendering

## SSR

Renderiza HTML en servidor por request. Mejora SEO y primera carga.

## SSG

Genera HTML estático en build. Bueno para contenido estable.

## Hydration

Reutiliza DOM del servidor en cliente para evitar recrear todo.

## Event replay

Guarda interacciones antes de que la app hidrate completamente.

## Ejemplo


```ts
bootstrapApplication(AppComponent, {
  providers: [provideClientHydration(withEventReplay())]
});
```


## Criterio

Admin privado muchas veces no necesita SSR. Ecommerce, landing y contenido indexable sí pueden beneficiarse.

## Errores comunes

- Usar APIs browser-only en servidor
- No probar SSR build
- Agregar SSR sin necesidad de producto

## Práctica

- [ ] Decidir CSR/SSR/SSG para tres tipos de app

## Relacionado

- [[Arquitectura Performance Seguridad Angular]]
- [[Angular - Roadmap]]

## Decisión técnica

En **SSR Hydration y Rendering**, no busques una receta única. Angular senior consiste en elegir estructura según tamaño del equipo, vida útil del producto, cantidad de features, necesidad de SEO, performance, permisos y testing.

## Preguntas antes de decidir

- ¿Este código cambiará por negocio o por infraestructura?
- ¿La feature tiene ownership claro?
- ¿Necesita estado compartido o solo local?
- ¿Hay rutas lazy y boundaries?
- ¿Qué parte debe poder testearse sin levantar toda la app?

## Criterio de madurez

Una arquitectura buena no se ve solo en carpetas bonitas. Se ve cuando agregar una feature no rompe otra, cuando una PR es revisable, cuando los tests cubren riesgos y cuando el equipo entiende dónde poner cada pieza.
