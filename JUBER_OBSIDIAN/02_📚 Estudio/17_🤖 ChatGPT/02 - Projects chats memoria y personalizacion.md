---
tags: [ChatGPT, Projects, Memoria, Personalizacion]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Projects, chats, memoria y personalización

## Projects

Agrupan chats, archivos, instrucciones y contexto de un objetivo. Para programación, crea uno por producto o repositorio importante.

### Qué guardar

- propósito, usuarios y glosario;
- arquitectura vigente;
- contratos/API;
- reglas de repositorio;
- decisiones activas;
- archivos de referencia estables;
- chats separados por feature, bug o investigación.

### Qué no guardar

- logs gigantes que ya no importan;
- varias versiones contradictorias;
- secrets;
- toda la carpeta `node_modules` o artefactos de build;
- discusiones descartadas como si fueran decisiones.

## Chats

Usa un chat por línea de trabajo. Mantén el contexto acumulado cuando el objetivo y decisiones siguen vigentes; abre otro cuando cambie el problema o la conversación esté contaminada.

Patrón útil:

- Chat 1: mapa del repo.
- Chat 2: especificación/ADR.
- Chat 3: debugging concreto.
- Chat 4: revisión y cierre.

## Memoria

Sirve para preferencias o información estable que conviene recordar entre sesiones. No la uses como repositorio de verdad técnica: la arquitectura debe vivir en archivos versionados.

Guarda en memoria:

- lenguaje de respuesta;
- nivel de detalle;
- stacks habituales;
- preferencias de formato.

Evita guardar:

- secretos;
- estado temporal del branch;
- versiones que cambian;
- reglas que solo aplican a un repo.

## Personalización

Distribuye instrucciones por alcance:

| Alcance | Lugar |
|---|---|
| Todos tus chats | Personalización/memoria |
| Un producto | Project |
| Un repositorio Codex | `AGENTS.md`/rules |
| Una tarea | Prompt actual |
| Un procedimiento repetido | Skill |

## Proyecto ideal para programación

```text
MiProducto/
  instrucciones: stack, arquitectura, validación y límites
  conocimiento: README, ADRs, OpenAPI, esquema, convenciones
  chats:
    00 mapa y onboarding
    01 decisiones
    bug - login 401
    feature - paginación
    review - release 2.4
```

## Trucos de aprovechamiento

- Fecha los documentos y declara cuál es canónico.
- Pide citar archivo/sección antes de una conclusión.
- Crea un handoff limpio al cerrar una conversación larga.
- Mueve reglas estables al Project o repo; no las repitas en cada prompt.
- Revisa periódicamente memoria e instrucciones obsoletas.

Fuentes: [Projects y chats](https://learn.chatgpt.com/docs/projects) y [personalización](https://learn.chatgpt.com/docs/personalize).

