# (Command Query Responsibility Segregation)

## Qué es
Patrón arquitectónico que separa las operaciones de lectura y escritura en modelos distintos.
## Principio
- **Commands** → modifican estado, no retornan datos
- **Queries** → retornan datos, no modifican estado
## Beneficios
- Escalar lecturas y escrituras de forma independiente
- Optimizar consultas sin afectar el modelo de dominio
- Separar lógica compleja de negocio (escritura) de consultas simples (lectura)
## Cuándo usarlo
- Alta disparidad entre volumen de lecturas vs escrituras
- Consultas que requieren estructuras muy diferentes al modelo de escritura
- Dominios complejos
## Cuándo NO usarlo
- CRUDs simples
- Proyectos pequeños donde añade complejidad innecesaria

---
# MediatR

## Qué es
Librería que implementa el patrón Mediator en .NET.

## Cómo funciona
1. Defines un **Request** (Command o Query) con los datos
2. Defines un **Handler** con la lógica
3. Inyectas `IMediator` y llamas `Send()`, MediatR encuentra el handler automáticamente
## Beneficios
- Desacopla controllers de lógica de negocio
- Pipeline behaviors (validación, logging, transacciones) sin ensuciar handlers
- Organización clara de responsabilidades
## Componentes principales
| Componente | Descripción |
|------------|-------------|
| `IRequest<T>` | Define el comando/query y su tipo de retorno |
| `IRequestHandler<TRequest, TResponse>` | Procesa el request |
| `IPipelineBehavior<TRequest, TResponse>` | Middleware del pipeline |

---
# Relación entre ambos

> [!note] Concepto clave
> **CQRS** es el patrón (el *qué*), **MediatR** es una herramienta que facilita implementarlo (un *cómo*). Puedes hacer CQRS sin MediatR.