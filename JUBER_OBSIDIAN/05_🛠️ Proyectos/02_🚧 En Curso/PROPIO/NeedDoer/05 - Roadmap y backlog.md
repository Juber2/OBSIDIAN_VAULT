---
tipo: roadmap
estado: borrador
actualizado: 2026-10-17
tags:
  - needdoer
  - roadmap
  - backlog
---

# Roadmap y backlog

Volver a [[NeedDoer]].

Este roadmap agrupa el mapa funcional para facilitar la planificación. La prioridad oficial de cada función se mantiene en [[02 - Mapa de funcionalidades]].

## Infraestructura del prototipo — Fase 1

- [x] Crear el monorepo local con npm workspaces.
- [x] Configurar React, TypeScript, Vite y Tailwind CSS.
- [x] Configurar Node.js, Express y variables de entorno.
- [x] Configurar Prisma y SQLite con una migración inicial.
- [x] Implementar la comprobación web → API → base de datos.
- [x] Crear scripts de arranque, compilación y comprobación de tipos.
- [x] Documentar instalación, arquitectura y comandos.
- [x] Verificar compilación, tipos, auditoría y presentación responsive.

## Prototipo — Etapa actual

El objetivo es validar la propuesta mediante un sistema local funcional, sin desplegar todavía infraestructura de producción.

- [x] Crear una navegación sencilla entre cliente, prestador y administración.
- [x] Representar la app de cliente y prestador con formato móvil.
- [x] Representar el panel administrativo con formato de escritorio.
- [x] Implementar publicación, cotización, aceptación, consumo de crédito y seguimiento del trabajo.
- [x] Crear una API local para compartir los datos entre los tres modos.
- [x] Usar una base de datos SQLite con datos de prueba reiniciables.
- [x] Simular únicamente los servicios externos: SMS, pagos, mapas, push y verificación de identidad.
- [x] Permitir reiniciar la demostración y recuperar los datos iniciales.
- [ ] Recoger cambios de funcionalidades después de revisar el prototipo.

React se usará para la interfaz, con un servicio local ligero y SQLite para persistencia. La tecnología del producto final se decidirá más adelante.

## Etapa 0 — Cerrar la definición

- [ ] Validar actores, categorías y ciudad de lanzamiento.
- [ ] Aprobar el flujo y los estados de la solicitud.
- [ ] Revisar el MVP función por función.
- [ ] Resolver las decisiones que cambian arquitectura, costos o cumplimiento legal.
- [ ] Definir métricas del lanzamiento.

## MVP — Primera versión operativa

### Base de la plataforma

- [ ] Registro por celular, datos básicos, consentimiento, direcciones y eliminación de cuenta.
- [x] Cambio entre modo cliente y modo prestador.
- [ ] Chat con texto, fotos, estados, reporte, bloqueo y cierre.
- [ ] Notificaciones push y bandeja dentro de la app.
- [ ] Calificación mutua con publicación a ciegas.
- [ ] Centro de ayuda, contacto de soporte y ajustes.

### Experiencia del cliente

- [ ] Inicio y publicación guiada de solicitudes.
- [ ] Fotos, ubicación aproximada, fecha, presupuesto y resumen.
- [ ] Lista y detalle de solicitudes por estado.
- [ ] Comparación de cotizaciones y perfiles, elección del prestador y chat durante el servicio activo.
- [ ] Seguimiento del trabajo y confirmación de finalización.
- [ ] Reportes, historial y calificación del prestador.

### Experiencia del prestador

- [ ] Alta, especialidades, zona de trabajo y verificación manual.
- [ ] Perfil público, portafolio y vista previa.
- [ ] Disponibilidad, solicitudes relevantes, filtros y detalle.
- [ ] Envío, seguimiento y retiro de cotizaciones sin consumo anticipado.
- [ ] Agenda, navegación, estados, monto final y cancelación.
- [ ] Saldo, compra, movimientos, comprobantes, créditos de bienvenida y consumo automático al ser elegido.

### Operación y administración

- [ ] Dashboard de operación y ventas.
- [ ] Verificación de prestadores.
- [ ] Gestión de usuarios, solicitudes y trabajos.
- [ ] Reportes, reclamos y notas de resolución.
- [ ] Catálogo de categorías e íconos.
- [ ] Paquetes, precios, ajustes y reembolsos de créditos.

## Fase 2 — Después de validar el MVP

- [ ] Acceso con Google y Apple.
- [ ] Respuestas rápidas, notas de voz y preferencias de notificación.
- [ ] Buscador, preguntas guiadas y videos cortos.
- [ ] Favoritos, solicitudes directas y repetición de solicitudes.
- [ ] Certificados, estudios, comprobantes y métricas públicas del prestador.
- [ ] Horarios, mapa de solicitudes y plantillas para cotizar.
- [ ] Bonos, referidos y estadísticas privadas.
- [ ] Moderación de reseñas, promociones y reglas configurables.
- [ ] Contenido administrable, roles del equipo y auditoría.

## Preparación de cada funcionalidad

Antes de pasar una función a desarrollo debe tener:

- Historia de usuario y resultado esperado.
- Criterios de aceptación y casos de error.
- Pantallas o flujo relacionados.
- Datos necesarios y reglas aplicables.
- Eventos de notificación.
- Requisitos de administración, soporte y analítica.
- Dependencias externas y consideraciones legales.

## Seguimiento sugerido

| Estado | Uso |
| --- | --- |
| Idea | Aún no revisada. |
| Definición | Se están aclarando reglas y experiencia. |
| Lista | Tiene criterios de aceptación y dependencias resueltas. |
| En desarrollo | Se está implementando. |
| En validación | Está en pruebas funcionales. |
| Hecha | Cumple los criterios acordados. |

