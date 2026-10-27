---
tags: [Claude, Skills, CustomSkills, Automatizacion]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Skills y custom skills

## Qué es una Skill

Paquete de conocimiento/instrucciones y, opcionalmente, recursos/código para una tarea especializada. Claude puede activarla cuando es relevante o tú puedes seleccionarla/invocarla según superficie.

## Skills incorporadas

Claude ofrece skills para creación/procesamiento profesional de Excel, Word, PowerPoint y PDF cuando code execution está habilitado.

Programación:

- informe de auditoría en Word/PDF;
- matriz de tests en Excel;
- presentación de arquitectura;
- análisis de logs y reporte ejecutivo.

## Custom Skills

Buenas candidatas:

- review ASP.NET Core;
- generación de ADR;
- migración EF Core;
- checklist Angular accessible;
- release/runbook;
- debugging de incidentes;
- formato de PR corporativo.

## Anatomía

```text
skill/
  SKILL.md       # propósito, trigger, instrucciones, límites
  references/    # normas/templates cargados cuando apliquen
  scripts/       # validación o transformación opcional
  assets/        # plantillas reutilizables
```

## Diseño correcto

- un problema repetible;
- descripción específica para activación;
- inputs/outputs claros;
- pasos verificables;
- ejemplos solo si aportan;
- seguridad y acciones externas explícitas;
- recursos cargados progresivamente.

## Skill vs instrucciones

| Necesidad | Usa |
|---|---|
| Preferencia global | Profile |
| Contexto de Project | Project instructions |
| Regla Code siempre activa | `CLAUDE.md` |
| Proceso reusable | Skill |
| Ejecución determinista | Hook/script |

## Skill + Connector

Ejemplo: GitHub connector obtiene PR; skill define severidades, revisión de seguridad, formato y criterios. El conector aporta datos; la skill evita un review genérico.

## Seguridad

Una Skill puede contener o instalar software. Revisa source, scripts, dependencias y side effects. Desactiva skills no usadas; no permitas activación automática en flujos con escritura peligrosa.

Fuentes: [usar Skills](https://support.claude.com/en/articles/12512180-use-skills-in-claude) y [crear custom Skills](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills).

