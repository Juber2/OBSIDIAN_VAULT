---
tags:
  - Java
  - Arquitectura
Creado: 2026-05-21
Relacionado:
  - Arquitectura Java
  - Microservicios
---

# Modular Monolith

Un monolito modular es una aplicacion desplegada como una sola unidad, pero dividida internamente por modulos de negocio.

## Por que importa

Muchas empresas saltan a microservicios antes de tener limites claros. Un monolito modular permite mantener simplicidad operativa mientras preparas el sistema para crecer.

## Estructura

```text
com.app
  sales/
    domain/
    application/
    infrastructure/
  billing/
  inventory/
  shared/
```

## Reglas

- Un modulo no debe acceder directamente a tablas internas de otro modulo.
- La comunicacion entre modulos debe pasar por interfaces, eventos o servicios publicos.
- `shared` debe ser pequeno. Si todo termina ahi, el diseno fallo.

## Spring Modulith

Spring Modulith ayuda a validar dependencias entre modulos, publicar eventos de dominio y documentar la arquitectura interna.

## Checklist

- [ ] Los paquetes representan capacidades de negocio.
- [ ] Hay limites claros entre modulos.
- [ ] Evito dependencias circulares.
- [ ] Puedo extraer un modulo a microservicio si el negocio lo exige.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

