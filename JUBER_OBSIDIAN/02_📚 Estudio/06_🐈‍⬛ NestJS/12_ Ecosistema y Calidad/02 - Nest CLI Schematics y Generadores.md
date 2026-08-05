---
tags:
  - NestJS
  - TypeScript
  - Nest-Calidad
Creado: 2026-07-27
Relacionado:
  - Ecosistema y Calidad
  - NESTJS
---

# 02 - Nest CLI Schematics y Generadores

## Que vas a aprender

En este capitulo vas a profundizar en el Nest CLI mas alla de lo basico (ya visto en fundamentos): **schematics personalizados** para generar codigo siguiendo las convenciones especificas de tu equipo, y monorepos con `nest-cli.json`.

## Explicacion

Los schematics son plantillas de generacion de codigo; Nest los usa internamente para `nest generate module/controller/service`, pero tambien permite crear **schematics propios** que generen la estructura exacta que tu equipo usa (por ejemplo, un modulo completo con controller, servicio, DTOs, tests, y hasta un archivo de documentacion, todo en un solo comando). Esto reduce errores de "me olvide de crear el test" o inconsistencias entre como distintos desarrolladores estructuran un modulo nuevo.

Ademas, `nest-cli.json` permite configurar un **monorepo** con multiples aplicaciones y librerias compartidas dentro del mismo repositorio Nest, util cuando varios servicios comparten DTOs, utilidades o logica de dominio.

## Modelo mental

Pensa un schematic personalizado como un molde de galletas hecho a medida: en vez de que cada desarrollador arme la estructura de un modulo nuevo a mano (con margen de error e inconsistencia), el molde genera siempre la misma forma correcta, alineada a la convencion del equipo.

## Ejemplo base

```bash
# Generar un modulo completo con recursos CRUD y tests incluidos
nest generate resource productos --type rest

# nest-cli.json para monorepo
{
  "projects": {
    "api": { "type": "application", "root": "apps/api" },
    "shared": { "type": "library", "root": "libs/shared" }
  }
}
```

## Como se usa en un proyecto real

1. Se usa `nest generate resource` para bootstrapear un CRUD completo (controller, servicio, DTOs, entidad, tests) en un solo comando.
2. Se evalua crear un schematic custom cuando el equipo repite frecuentemente la misma estructura no cubierta por los generadores default.
3. En proyectos con multiples servicios relacionados, se evalua un monorepo Nest con librerias compartidas (`libs/`) para DTOs y utilidades comunes.
4. Se documenta el uso de schematics custom en el README del proyecto para que nuevos integrantes los descubran facilmente.
5. Se evita crear schematics para casos que ocurren una unica vez; el beneficio aparece con repeticion.

## Criterio junior

Un junior debe poder usar `nest generate resource` para crear un CRUD completo siguiendo las convenciones default del CLI.

## Criterio mid

Un mid debe configurar un monorepo Nest basico con una aplicacion y una libreria compartida, entendiendo como se referencian entre si.

## Criterio senior

Un senior debe evaluar si un schematic custom aporta valor real al equipo, diseñar la estructura de un monorepo si el numero de servicios lo justifica, y balancear la ganancia de consistencia contra el costo de mantener herramientas de generacion propias.

## Errores comunes

- Generar recursos con el CLI y despues reestructurarlos manualmente de forma inconsistente entre distintos desarrolladores, perdiendo el beneficio de la convencion.
- Crear un monorepo prematuramente para un solo servicio, sumando complejidad de configuracion sin necesidad real.
- No documentar schematics custom, dejando que solo quien los creo sepa que existen y como usarlos.
- Compartir codigo entre servicios copiando y pegando en vez de extraerlo a una libreria compartida del monorepo.

## Practica

- [ ] Genera un recurso CRUD completo con `nest generate resource`.
- [ ] Configura un monorepo Nest simple con una aplicacion y una libreria compartida.
- [ ] Evalua si tu equipo se beneficiaria de un schematic custom para algun patron que repiten frecuentemente.
- [ ] Documenta en un README como usar los generadores y schematics de tu proyecto.
- [ ] Anota cuando NO valdria la pena migrar a un monorepo.

## Preguntas de repaso

- Que ventaja da `nest generate resource` sobre crear cada archivo de un CRUD manualmente?
- Cuando se justifica crear un schematic personalizado para tu equipo?
- Que beneficio da un monorepo con librerias compartidas sobre copiar codigo entre servicios?
- Que riesgo tiene migrar a un monorepo antes de necesitarlo realmente?
