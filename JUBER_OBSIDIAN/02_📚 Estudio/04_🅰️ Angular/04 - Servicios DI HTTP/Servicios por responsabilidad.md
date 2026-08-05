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

# Servicios por responsabilidad

## Tipos de servicio

| Tipo | Responsabilidad |
|---|---|
| API service | Llamar backend |
| Store service | Estado de feature |
| Facade | API simple para componentes |
| Utility | Funciones puras compartidas |

## Ejemplo API service


```ts
@Injectable({ providedIn: 'root' })
export class UsersApiService {
  private http = inject(HttpClient);

  getUsers() {
    return this.http.get<UserDto[]>('/api/users');
  }
}
```


## Regla práctica

Si un servicio hace HTTP, no debería decidir cómo se renderiza la pantalla. Si maneja estado, no debería conocer detalles de DOM.

## Errores comunes

- Un servicio llamado utils con todo dentro
- Servicios con 20 métodos no relacionados

## Práctica

- [ ] Separar UsersApiService y UsersStore

## Relacionado

- [[Servicios DI HTTP Angular]]
- [[Angular - Roadmap]]

## Diseño aplicado

En **Servicios por responsabilidad**, el punto importante es separar responsabilidades. Angular permite inyectar casi cualquier cosa, pero eso no significa que un servicio deba concentrar API, estado, reglas de negocio, navegación y mensajes visuales.

## Separación recomendada

- API service: habla con backend y devuelve DTOs o modelos mapeados.
- Store/facade de feature: expone estado y acciones para componentes.
- Adapter/mapper: traduce DTO externo a modelo interno.
- Utility: función pura sin dependencia de Angular.

## Ejemplo de criterio

Si una función necesita HttpClient, va hacia data-access. Si necesita Router, probablemente pertenece a una capa de flujo o componente contenedor. Si solo transforma UserDto en User, debería ser una función pura testeable.

## Revisión rápida

Abre el servicio y pregunta: ¿por qué cambia este archivo? Si cambia por demasiadas razones, está acumulando responsabilidades.
