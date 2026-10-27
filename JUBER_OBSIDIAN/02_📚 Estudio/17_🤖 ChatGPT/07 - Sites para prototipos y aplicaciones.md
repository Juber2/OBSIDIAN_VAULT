---
tags: [ChatGPT, Sites, Frontend, Deploy]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Sites para prototipos y aplicaciones

Sites permite crear, guardar, desplegar y administrar sitios alojados desde ChatGPT web o Desktop.

## Cuándo usarlo

- landing o micrositio;
- demo funcional para feedback;
- dashboard interno;
- prototipo de flujo;
- recurso educativo/interactivo;
- validación de una idea antes del repositorio definitivo.

## Cuándo no usarlo automáticamente

- backend regulado o datos sensibles sin arquitectura revisada;
- aplicación que exige infraestructura corporativa específica;
- reemplazar un repo/producto existente sin plan de migración;
- deploy que todavía no fue revisado.

## Flujo correcto

1. Describe audiencia, propósito, comportamiento y fuentes.
2. Pide guardar una versión **sin desplegar** para revisión.
3. Revisa contenido, datos, permisos, responsive y accesibilidad.
4. Prueba estados y enlaces.
5. Despliega solo cuando aceptes que la URL es producción.
6. Conserva/exporta el código cuando necesites continuidad fuera de Sites.

> [!warning]
> La documentación oficial indica que cada URL desplegada de Sites es un deployment de producción. No uses deploy como preview.

## Combinar con Codex

- Codex CLI/IDE: editar y probar el proyecto local.
- Sites en web/Desktop: guardar, desplegar y administrar.
- Browser/computer use: hacer QA del deployment.
- Appshots: documentar antes/después.

## Prompt de sitio

```text
Crea un website interno para visualizar salud de servicios. Antes de desplegar,
guarda una versión revisable. Debe tener datos simulados, filtros, estado vacío/error,
responsive, teclado y contraste AA. No uses información real ni servicios externos.
Incluye una lista de lo que faltaría para producción corporativa.
```

## Checklist previo a desplegar

- [ ] Datos y enlaces revisados.
- [ ] Sin secretos/PII.
- [ ] Responsive y accesible.
- [ ] Estados de error.
- [ ] Dependencias/orígenes comprendidos.
- [ ] Política de publicación aceptada.
- [ ] Propietario y mantenimiento definidos.

Fuente: [Sites – ChatGPT](https://learn.chatgpt.com/docs/sites).

