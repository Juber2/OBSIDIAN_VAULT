# Regla de negocio — Plantillas de cursos

## Pregunta pendiente de definir con el cliente

Cuando se **edita una plantilla de curso** que ya fue asignada a una o más empresas:
- ¿La edición afecta a las empresas que ya tienen ese curso asignado?
- ¿O cada empresa trabaja con una copia independiente?

---

## Opciones analizadas

### Opción A — Copia al asignar (snapshot) ✅ Recomendada
Al asignar la plantilla a una empresa se crea una **copia independiente**.  
Editar la plantilla original **no afecta** a las empresas que ya la tienen.

**Ventajas:**
- Cada empresa puede personalizar su propia versión
- No hay riesgo de romper cursos activos por un cambio en la plantilla

**Desventajas:**
- Si se corrige un error en la plantilla, hay que reasignar manualmente a cada empresa

---

### Opción B — Referencia viva
Las empresas **apuntan a la plantilla**. Si se edita, todas las empresas ven el cambio automáticamente.

**Ventajas:**
- Actualizaciones centralizadas

**Desventajas:**
- Un cambio accidental afecta a todas las empresas a la vez
- Problemático si hay empleados haciendo el curso en ese momento

---

## Estado
- [ ] Confirmar con el cliente cuál modelo aplicar
- [ ] Verificar cómo está implementado el `Assign` actualmente en el backend
