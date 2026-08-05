---
tags:
  - angular
  - laboratorio
  - forms-http
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos Angular
  - ANGULAR
---

# Laboratorio 03 - Routing forms y HTTP

## Objetivo

Crear navegación con detalle, formulario typed reactive y consumo HTTP tipado con DTO mapping.

## Rutas

`/users`, `/users/new`, `/users/:id/edit`. Usa route params para recurso y query params para filtros.

## Formulario

Typed Reactive Form con validaciones sincrónicas, mensajes por campo y bloqueo de submit mientras guarda.

## Servicio HTTP


```ts
getUsers(params: UserSearchParams): Observable<User[]> {
  return this.http.get<UserDto[]>("/api/users", { params }).pipe(
    map(dtos => dtos.map(toUser))
  );
}
```


## Errores comunes

- HttpClient directo en componente
- Formularios sin tipos
- No diferenciar 400/401/403/404

## Practica

- [ ] Crear UserForm typed
- [ ] Crear UsersApiService
- [ ] Mapear DTO a modelo

## Relacionado

- [[Laboratorios Practicos Angular]]
- [[ANGULAR]]

## Desarrollo guiado

Este laboratorio no se debe resolver copiando snippets aislados. La idea es construir una pequeña pieza funcional, ejecutar la app y registrar qué decisiones tomaste. Para **Laboratorio 03 - Routing forms y HTTP**, trabaja en tres capas: primero haz que funcione, luego ordena responsabilidades y finalmente agrega validación o prueba mínima.

## Pasos sugeridos

1. Crea una rama o carpeta de práctica con nombre claro.
2. Implementa el flujo mínimo sin optimizar antes de tiempo.
3. Separa UI, estado y acceso a datos si el ejercicio lo justifica.
4. Agrega al menos un caso de error: API falla, formulario inválido, sesión expirada o lista vacía.
5. Escribe una nota corta con lo que aprendiste y lo que cambiarías.

## Criterio de revisión

El laboratorio está completo cuando otra persona puede abrir el código, entender el flujo y ejecutarlo. Revisa nombres, estados de UI, manejo de errores y que no hayas escondido reglas importantes dentro del template.
