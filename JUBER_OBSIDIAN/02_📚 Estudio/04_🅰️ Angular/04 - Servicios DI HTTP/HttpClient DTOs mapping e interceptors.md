---
tags:
  - Angular
  - Intermedio
  - Servicios
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Servicios DI HTTP Angular
  - ANGULAR
---

# HttpClient DTOs mapping e interceptors

## HttpClient

Devuelve Observables y permite tipar respuestas, params, headers y body.

## DTO mapping

Convierte datos externos al modelo que usa tu UI. No dejes que toda la app dependa del formato backend.

## Ejemplo mapping


```ts
getUsers(): Observable<User[]> {
  return this.http.get<UserDto[]>('/api/users').pipe(
    map(dtos => dtos.map(toUser))
  );
}
```


## Interceptor funcional


```ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthStore).token();
  const authReq = token
    ? req.clone({ setHeaders: { Authorization: 'Bearer ' + token } })
    : req;

  return next(authReq);
};
```


## Manejo de errores

Diferencia 400, 401, 403, 404 y 500. No muestres el mismo mensaje para todo.

## Errores comunes

- Tipar response como any
- Duplicar URLs en componentes
- Un interceptor que redirige, loguea, transforma y maneja loading todo junto

## Práctica

- [ ] Crear servicio HTTP con mapper y error handling

## Relacionado

- [[Servicios DI HTTP Angular]]
- [[Angular - Roadmap]]

## Diseño aplicado

En **HttpClient DTOs mapping e interceptors**, el punto importante es separar responsabilidades. Angular permite inyectar casi cualquier cosa, pero eso no significa que un servicio deba concentrar API, estado, reglas de negocio, navegación y mensajes visuales.

## Separación recomendada

- API service: habla con backend y devuelve DTOs o modelos mapeados.
- Store/facade de feature: expone estado y acciones para componentes.
- Adapter/mapper: traduce DTO externo a modelo interno.
- Utility: función pura sin dependencia de Angular.

## Ejemplo de criterio

Si una función necesita HttpClient, va hacia data-access. Si necesita Router, probablemente pertenece a una capa de flujo o componente contenedor. Si solo transforma UserDto en User, debería ser una función pura testeable.

## Revisión rápida

Abre el servicio y pregunta: ¿por qué cambia este archivo? Si cambia por demasiadas razones, está acumulando responsabilidades.
