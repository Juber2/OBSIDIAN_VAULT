---
tags:
  - dotnet
  - Intermedio
  - seguridad
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Seguridad Auth JWT dotNET
  - NET
---

# Policies RBAC permisos y recursos

## RBAC

Roles como SuperAdmin, CompanyAdmin y Employee sirven para reglas gruesas.

## Policies

Permiten expresar requisitos: claim, rol, tenant, permiso o handler custom.

## Resource based authorization

Cuando el permiso depende del recurso: solo dueño del curso, admin del tenant, creador del registro.

## Ejemplo policy


```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("TenantAdmin", policy =>
        policy.RequireClaim("userType", "tenant")
              .RequireRole("CompanyAdmin"));
});
```


## Errores comunes

- Roles explosivos para cada accion
- Validar ownership en frontend
- No auditar acciones sensibles

## Practica

- [ ] Crear policy para platform y tenant admin

## Relacionado

- [[Seguridad Auth JWT dotNET]]
- [[NET - Roadmap]]
