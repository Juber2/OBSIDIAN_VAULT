---
tags: [Claude, Memoria, ChatSearch, Personalizacion]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Memoria, chat search e instrucciones

## Chat search

Permite pedirle a Claude que encuentre conversaciones anteriores y use contexto relevante. Es útil para recuperar decisiones, explicaciones o una investigación previa.

No lo uses como sustituto de Git/ADRs: una decisión de producción debe vivir en una fuente versionada.

## Memoria

Genera continuidad entre chats según experiencia/plan. Guarda información estable:

- stacks y nivel técnico;
- idioma/formato;
- proyectos habituales;
- preferencias de colaboración.

Evita:

- secrets;
- estado actual del branch;
- tickets temporales;
- decisiones que deben auditarse;
- reglas que aplican a un solo repositorio.

## Incognito

Úsalo cuando no quieres que una conversación contribuya a memoria/continuidad, por ejemplo un cliente aislado o una exploración temporal. Sigue aplicando políticas de datos: incognito no autoriza compartir material confidencial.

## Profile vs Project vs Skill vs `CLAUDE.md`

| Necesidad | Lugar |
|---|---|
| Preferencia global | Profile/memory |
| Contexto de producto | Project instructions |
| Capacidad reusable | Skill |
| Reglas de repositorio | `CLAUDE.md` |
| Requisito actual | Prompt |

## Auditoría

Cada mes revisa memoria e instrucciones:

- ¿sigue vigente?
- ¿es demasiado específica?
- ¿se contradice con Project?
- ¿debe convertirse en documento/skill?
- ¿contiene datos que conviene eliminar?

## Uso práctico

```text
Busca en mis chats anteriores la decisión sobre paginación del proyecto Cursos.
Indica fecha y conversación, pero no la trates como vigente hasta contrastarla con
el ADR y OpenAPI actuales del Project.
```

Fuentes: [chat search y memoria](https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context) y [personalización](https://support.claude.com/en/articles/10185728-understanding-claude-s-personalization-features).

