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

# Dependency Injection e InjectionToken

## DI

Angular crea y entrega dependencias mediante providers. Eso evita `new` disperso y mejora testabilidad.

## inject()

Forma moderna de pedir dependencias, especialmente útil en guards, interceptors y servicios.

## InjectionToken

Sirve para valores o contratos que no son clases.

## Ejemplo


```ts
export const API_URL = new InjectionToken<string>('API_URL');

export const appConfig: ApplicationConfig = {
  providers: [{ provide: API_URL, useValue: 'https://api.app.com' }]
};

const apiUrl = inject(API_URL);
```


## Jerarquía

Angular busca providers desde el injector local hacia arriba. Un provider en componente crea instancia para ese subárbol.

## Errores comunes

- Providers duplicados sin querer
- Config hardcodeada
- Inyectar implementaciones concretas donde conviene token

## Práctica

- [ ] Crear token API_URL y usarlo en servicio

## Relacionado

- [[Servicios DI HTTP Angular]]
- [[Angular - Roadmap]]

## Diseño aplicado

En **Dependency Injection e InjectionToken**, el punto importante es separar responsabilidades. Angular permite inyectar casi cualquier cosa, pero eso no significa que un servicio deba concentrar API, estado, reglas de negocio, navegación y mensajes visuales.

## Separación recomendada

- API service: habla con backend y devuelve DTOs o modelos mapeados.
- Store/facade de feature: expone estado y acciones para componentes.
- Adapter/mapper: traduce DTO externo a modelo interno.
- Utility: función pura sin dependencia de Angular.

## Ejemplo de criterio

Si una función necesita HttpClient, va hacia data-access. Si necesita Router, probablemente pertenece a una capa de flujo o componente contenedor. Si solo transforma UserDto en User, debería ser una función pura testeable.

## Revisión rápida

Abre el servicio y pregunta: ¿por qué cambia este archivo? Si cambia por demasiadas razones, está acumulando responsabilidades.
