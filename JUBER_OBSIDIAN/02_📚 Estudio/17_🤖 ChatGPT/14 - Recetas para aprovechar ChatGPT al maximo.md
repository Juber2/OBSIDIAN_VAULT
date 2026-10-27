---
tags: [ChatGPT, Codex, Recetas, Prompts]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Recetas para aprovechar ChatGPT al máximo

El prompt es solo el pegamento entre herramientas. La receta importante es elegir la combinación correcta.

## 1. Feature desde diseño

```text
Figma/plugin → extraer diseño y tokens
Project → obtener reglas/producto
Codex IDE → implementar selección/feature
Browser → probar flujo
Appshot → comparar
Codex → corregir y ejecutar tests
```

Prompt mínimo:

```text
Implementa este diseño en el feature existente. Reutiliza componentes/tokens; incluye
loading, empty, error, responsive y teclado. Compara visualmente y ejecuta tests/build.
```

## 2. Bug difícil

```text
Files → logs/export
Visualization → timeline/patrón
Web search → docs/CVEs actuales
Codex → reproducir, test, fix
Browser/computer use → verificar conducta real
```

## 3. Migración de framework

```text
Research → guía oficial/deprecaciones
Project → arquitectura y restricciones
Codex cloud → lotes paralelos en worktrees
CLI → validación completa
Scheduled task → seguimiento de pendientes
```

## 4. Code review serio

```text
GitHub plugin → PR/diff
Skill → checklist del stack
Subagentes → seguridad/performance/tests
Codex code review → hallazgos
Humano → valida escenarios antes de comentar
```

## 5. Aprender una tecnología

```text
Web → docs oficiales
Project → ruta y archivos
Visualization → modelos mentales
Codex → laboratorios ejecutables
Scheduled task → práctica espaciada
```

## 6. Mantenimiento semanal

```text
Scheduled task + web search + skill de dependencias
→ reporte con changelogs/CVEs
→ aprobación humana
→ Codex cloud abre cambio aislado
→ CLI/CI verifica
```

## Prompt universal breve

```text
Objetivo: [resultado].
Fuentes: [Project/files/web/plugin].
Acciones permitidas: [leer/editar/testear].
Requiere aprobación: [dependencias/red/publicar/borrar].
Éxito: [criterios + comandos].
Entrega: [artefacto/diff/reporte].
```

## Checklist de selección

- [ ] ¿Necesito actualidad? Web.
- [ ] ¿Necesito contexto persistente? Project.
- [ ] ¿Necesito código real? Codex.
- [ ] ¿Necesito UI real? Browser/computer use.
- [ ] ¿Necesito servicio externo? Plugin/MCP.
- [ ] ¿Es recurrente? Skill/scheduled task.
- [ ] ¿Es paralelizable? Cloud/subagentes.
- [ ] ¿Se puede publicar? Sites, con revisión.
- [ ] ¿Tengo permisos mínimos y verificación?

Volver al índice: [[CHATGPT]].

