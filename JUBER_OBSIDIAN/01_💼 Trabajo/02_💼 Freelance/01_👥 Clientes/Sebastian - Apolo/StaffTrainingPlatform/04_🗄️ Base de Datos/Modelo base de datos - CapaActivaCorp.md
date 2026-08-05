## Como se gestionara los tipos de roles y usuarios

## ✅ Resumen 

### 1️⃣ Tipos de roles

Existen **2 tipos de roles**:

- **GLOBAL** → nivel plataforma
    
- **COMPANY** → nivel empresa
---
### 2️⃣ Rol GLOBAL (`identity.user.role_id`)

- Todo usuario **SIEMPRE** tiene un rol GLOBAL (nunca `null`).
    
- Ejemplos:
    
    - `SUPERADMIN`
        
    - `ADMIN_PARCIAL`
        
    - `USER_CLIENT`
        
- El rol GLOBAL:
    
    - identifica el tipo de cuenta
        
    - da permisos mínimos de plataforma
        
    - **NO da permisos dentro de empresas**
---
### 3️⃣ Rol COMPANY (`identity.user_company.role_id`)

- Define **qué puede hacer el usuario dentro de una empresa específica**.
- Ejemplos:
    
    - `ADMIN_EMPRESA`
        
    - `EDITOR`
        
    - `EMPLEADO_COMPANY`
        
    - `OBSERVADOR`
        
- Un usuario puede tener **distintos roles COMPANY en distintas empresas**.
---
### 4️⃣ `user_company`

- Representa la relación **usuario ↔ empresa**.
    
- Un usuario puede tener **muchos registros** (una por empresa).
    
- `(user_id, company_id)` es único.
    
- Aquí vive el **rol real operativo** del usuario.
---
### 5️⃣ Usuarios de empresa (cliente y empleados)

- El **admin del cliente** y sus **empleados**:
    - tienen el **mismo rol GLOBAL** (ej. `USER_CLIENT`)
    - se diferencian solo por su **rol COMPANY** en `user_company`.

Ejemplo:
- Admin del cliente → `ADMIN_EMPRESA`
- Empleado → `EMPLEADO_COMPANY`
---
### 6️⃣ Regla clave (la más importante)

> **El rol GLOBAL define “qué es el usuario en la plataforma”.**  
> **El rol COMPANY define “qué puede hacer dentro de una empresa”.**

Incluso un superadmin:
- necesita `user_company` para operar dentro de una empresa
- y usa el rol COMPANY para permisos de esa empresa