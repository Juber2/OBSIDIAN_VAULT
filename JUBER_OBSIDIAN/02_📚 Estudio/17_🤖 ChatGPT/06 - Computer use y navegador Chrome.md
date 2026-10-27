---
tags: [ChatGPT, ComputerUse, Browser, Chrome, QA]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Computer use y navegador Chrome

## Diferencias

| Herramienta | Opera sobre | Mejor uso |
|---|---|---|
| Browser | Página web controlada/inspeccionada | Navegar, probar, capturar evidencia |
| Chrome extension | Sesión existente del navegador | Trabajar con páginas donde ya estás autenticado |
| Computer use | Apps y pantalla del equipo | Flujos que cruzan varias interfaces |
| Connector/plugin | API estructurada | Leer/escribir datos con mayor precisión |

Prefiere una API/conector cuando existe. Computer use es útil cuando la única interfaz disponible es visual.

## QA de aplicaciones

Casos potentes:

- login/logout y recuperación;
- formularios y validación;
- navegación por teclado;
- responsive en viewports;
- carga, vacío, error y éxito;
- regresiones visuales;
- flujos que cruzan frontend y proveedor externo.

## Encargo de QA

```text
Prueba el flujo de creación de curso en el entorno local autorizado. Usa esta cuenta
de prueba. Ejecuta: feliz, campo requerido vacío, archivo inválido y pérdida de red.
No borres datos ajenos ni confirmes acciones irreversibles. Por fallo registra pasos,
resultado esperado/real, screenshot y severidad. No corrijas el código todavía.
```

## Aprovechamiento máximo

1. Entrega URL, credenciales de prueba y datos aislados.
2. Define acciones permitidas/prohibidas.
3. Pide screenshots por estado.
4. Convierte cada fallo en test automatizable.
5. Después pasa el informe a Codex para diagnosticar y corregir.
6. Repite el flujo tras el cambio.

## Riesgos

- La UI puede cambiar y romper la navegación.
- Un clic equivocado puede escribir datos reales.
- Texto externo puede intentar redirigir al agente.
- La sesión del navegador puede contener datos sensibles.
- CAPTCHA/MFA o confirmaciones requieren intervención humana.

Usa cuentas de prueba, mínimo privilegio y aprobación para publicar, comprar, enviar o borrar.

Fuentes: [computer use](https://learn.chatgpt.com/docs/computer-use), [browser](https://learn.chatgpt.com/docs/browser) y [Chrome extension](https://learn.chatgpt.com/docs/chrome-extension).

