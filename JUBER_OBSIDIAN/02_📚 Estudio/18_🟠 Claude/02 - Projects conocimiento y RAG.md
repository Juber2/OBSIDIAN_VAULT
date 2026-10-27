---
tags: [Claude, Projects, RAG, Contexto]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Projects, conocimiento y RAG

## Qué contiene un Project

- chats relacionados;
- instrucciones de proyecto;
- documentos, texto y fragmentos de código;
- base de conocimiento compartida.

Importante: el nombre/descripcion visible no garantiza que Claude use ese contenido como instrucciones. Configura **Project instructions** y conocimiento explícitamente.

## Para un proyecto de software

Incluye:

- README y comandos;
- arquitectura/ADRs;
- OpenAPI/contratos;
- esquema de datos;
- glosario de dominio;
- decisiones activas;
- style/design system;
- documentación de versiones.

No incluyas builds, dependencias vendorizadas, secrets, dumps ni múltiples versiones sin autoridad clara.

## RAG

Cuando el conocimiento se acerca al límite de contexto, Claude puede activar recuperación aumentada: busca y trae las partes relevantes en vez de cargar todo.

### Ventajas

- más capacidad documental;
- respuestas rápidas con corpus grande;
- no repetir uploads en cada chat.

### Límites prácticos

- puede recuperar el fragmento equivocado;
- documentos duplicados/obsoletos compiten;
- nombres pobres reducen descubribilidad;
- una respuesta debe citar o identificar fuente.

## Cómo estructurar para RAG

```text
knowledge/
  00-glossary.md
  architecture-current.md
  adr-001-auth.md
  api-openapi-2026-08.yaml
  database-schema-v14.sql
  testing-guide.md
  legacy/ (marcado como histórico)
```

## Prompts para anclar fuentes

```text
Antes de responder, localiza las fuentes del Project que definen autenticación.
Enumera archivo y sección. Si hay contradicción, detente y muéstrala. No uses una
decisión histórica como vigente.
```

## GitHub en Project

La integración puede añadir repositorios/archivos como contexto. Refresca la sincronización y confirma branch/commit antes de analizar. Para ejecutar cambios, usa Claude Code; integrar GitHub al chat no equivale automáticamente a terminal.

Fuentes: [crear y administrar Projects](https://support.claude.com/en/articles/9519177-how-can-i-create-and-manage-projects) y [RAG para Projects](https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects).

