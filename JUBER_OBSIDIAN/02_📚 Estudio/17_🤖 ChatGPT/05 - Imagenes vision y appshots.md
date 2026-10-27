---
tags: [ChatGPT, Vision, Imagenes, Appshots, Frontend]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Imágenes, visión y appshots

## Image inputs / visión

Adjunta screenshots, diagramas, diseños o errores visuales para que ChatGPT los interprete.

Usos de programación:

- convertir un diseño en requisitos de UI;
- detectar overflow, contraste o jerarquía deficiente;
- leer un diagrama legacy;
- comparar esperado vs actual;
- interpretar un error visible;
- analizar un plan o dashboard.

## Cómo adjuntar una captura útil

Incluye:

- resolución y viewport;
- navegador/dispositivo;
- estado exacto;
- qué esperabas;
- qué zona debe analizar;
- screenshot completo y recorte si hace falta.

No confíes en OCR perfecto: verifica textos, números y código pequeño.

## Image generation

Úsala para assets provisionales, mockups, ilustraciones, texturas o variantes visuales. No es la herramienta adecuada para editar SVG/código que ya pertenece al design system.

Para frontend:

- moodboard antes de código;
- placeholder de producto;
- fondos/hero temporales;
- comparación de estilos;
- prototipo para obtener feedback.

Comprueba licencia/política, resolución, accesibilidad y peso antes de producción.

## Appshots

Capturas estructuradas del estado de una app pueden servir como evidencia visual y contexto para iteraciones. Úsalas para:

- documentar un bug UI;
- comparar antes/después;
- guiar una revisión responsive;
- alinear diseño y código;
- registrar estados loading/empty/error.

## Flujo visual recomendado

```text
Referencia visual → especificación de UI → implementación Codex
→ ejecutar app → browser/computer use → appshot → comparación → ajuste
```

## Prompt útil

```text
Compara estas dos capturas a 1440×900. La primera es diseño aprobado y la segunda
es implementación. Enumera diferencias medibles en layout, tipografía, spacing,
color, estados y accesibilidad. No propongas cambios hasta priorizar por impacto.
```

Fuentes: [image inputs](https://learn.chatgpt.com/docs/image-inputs), [image generation](https://learn.chatgpt.com/docs/image-generation) y [appshots](https://learn.chatgpt.com/docs/appshots).

