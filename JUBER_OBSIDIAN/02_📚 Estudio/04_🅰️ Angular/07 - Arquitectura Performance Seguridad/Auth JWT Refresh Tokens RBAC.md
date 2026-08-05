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

# Auth JWT Refresh Tokens RBAC

## Responsabilidad frontend

Adjuntar token, manejar expiración visual, proteger rutas por UX y mostrar/ocultar acciones.

## Responsabilidad backend

Validar token, permisos, tenant, roles y reglas. El backend es autoridad real.

## Piezas Angular

AuthStore, interceptor, guard, manejo de 401/403, directivas de permisos si aplica.

## Flujo refresh

Si API devuelve `TOKEN_EXPIRED`, interceptor intenta refresh; si falla, limpia sesión y redirige al login.

## Guard ejemplo


```ts
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthStore);
  const router = inject(Router);
  return auth.isAuthenticated() ? true : router.createUrlTree(['/login']);
};
```


## Errores comunes

- Creer que ocultar botón es seguridad
- No diferenciar 401/403
- Guardar tokens sin estrategia

## Práctica

- [ ] Diseñar flujo login + refresh + logout

## Relacionado

- [[Arquitectura Performance Seguridad Angular]]
- [[Angular - Roadmap]]

## Decisión técnica

En **Auth JWT Refresh Tokens RBAC**, no busques una receta única. Angular senior consiste en elegir estructura según tamaño del equipo, vida útil del producto, cantidad de features, necesidad de SEO, performance, permisos y testing.

## Preguntas antes de decidir

- ¿Este código cambiará por negocio o por infraestructura?
- ¿La feature tiene ownership claro?
- ¿Necesita estado compartido o solo local?
- ¿Hay rutas lazy y boundaries?
- ¿Qué parte debe poder testearse sin levantar toda la app?

## Criterio de madurez

Una arquitectura buena no se ve solo en carpetas bonitas. Se ve cuando agregar una feature no rompe otra, cuando una PR es revisable, cuando los tests cubren riesgos y cuando el equipo entiende dónde poner cada pieza.
