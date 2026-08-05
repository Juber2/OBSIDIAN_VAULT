---
tags:
  - NestJS
  - TypeScript
  - Nest-Proyectos
Creado: 2026-07-27
Relacionado:
  - Proyectos
  - NESTJS
---

# 03 - Proyecto SaaS Multi Tenant

## Que vas a construir

Un backend multi-tenant simplificado (varios "clientes"/organizaciones aislados dentro de la misma aplicacion), aplicando arquitectura de monolito modular y decisiones de escalabilidad, sobre la base de auth ya construida en el proyecto anterior.

## Alcance funcional

- Modelo de tenant (organizacion) al que pertenecen los usuarios y los datos.
- Aislamiento de datos por tenant en cada query (nunca un usuario ve datos de otro tenant).
- Al menos dos modulos de dominio con limites claros (monolito modular, ver capitulo de Arquitectura).
- Un flujo de negocio multi-tabla protegido con transaccion (por ejemplo, crear una organizacion junto con su usuario administrador inicial).
- Cache de una consulta costosa y frecuente, con invalidacion correcta al escribir.

## Checklist tecnica

- [ ] Toda query que accede a datos de negocio filtra explicitamente por `tenantId`, sin excepciones olvidadas.
- [ ] Existe al menos un test que verifica que un usuario del tenant A no puede ver datos del tenant B.
- [ ] Los modulos de dominio no acceden a tablas de otros modulos directamente (limites de monolito modular respetados).
- [ ] La operacion multi-tabla critica esta envuelta en una transaccion con manejo correcto de fallo parcial.
- [ ] El cache implementado se invalida correctamente al modificar el dato subyacente.
- [ ] Se documenta con un ADR la decision de como se implemento el aislamiento multi-tenant (columna `tenantId` vs esquema separado vs base separada) y por que.

## Criterio de exito

El proyecto se considera completo cuando, con datos de dos tenants distintos cargados, es imposible (verificado con un test automatizado, no solo revision manual) que un usuario autenticado de un tenant acceda a datos del otro por ningun endpoint.

## Errores comunes a evitar

- Olvidar el filtro por `tenantId` en un endpoint nuevo, el error de aislamiento multi-tenant mas comun y mas peligroso.
- Confiar solo en revision manual para el aislamiento entre tenants, sin un test automatizado que lo verifique en cada cambio futuro.
- Mezclar responsabilidades entre modulos de dominio, rompiendo los limites del monolito modular.

## Extension opcional

Evaluar (sin necesariamente implementar) si este sistema justificaria migrar a microservicios en el futuro, documentando el razonamiento con el marco de decisiones arquitectonicas del capitulo Senior.
