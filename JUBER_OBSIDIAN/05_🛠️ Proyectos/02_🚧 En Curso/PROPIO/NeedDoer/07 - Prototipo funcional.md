---
tipo: definición de prototipo
estado: acordado
actualizado: 2026-10-20
tags:
  - needdoer
  - prototipo
  - arquitectura-local
---

# Prototipo funcional

Volver a [[NeedDoer]].

## Objetivo

Construir un prototipo local que permita utilizar los recorridos principales de NeedDoer como si fuera un sistema real. Todas las acciones incluidas en el prototipo deben guardar información y producir efectos visibles en los demás modos.

Por ejemplo, una solicitud publicada por un cliente aparecerá entre las oportunidades de los prestadores. Una cotización enviada aparecerá al cliente y, cuando este la acepte, se consumirá el crédito del prestador elegido. El trabajo también será visible y administrable desde el panel.

## Estado de implementación

- **Ruta local:** `C:\JUBER\FREELANCE\NEEDDOER`.
- **Fase actual:** Fases 1 a 12 completadas.
- **Última verificación:** 2026-10-17.
- **Rama:** `main` (la Fase 3 se fusionó desde `codex/fase-3-diseno-navegacion`).
- **Datos de demostración:** `npm run db:seed` los restablece; las cuentas de prueba están en el README del proyecto.
- **Arranque:** ejecutar `npm run dev` desde la raíz.
- **Aplicación:** `http://localhost:5173`.
- **API:** `http://localhost:4000/api`.

### Plan de implementación

| Fase | Alcance                                              | Estado    |
| ---- | ---------------------------------------------------- | --------- |
| 1    | Creación, configuración, React, API, Prisma y SQLite | Hecha     |
| 2    | Modelo de datos y datos de prueba                    | Hecha     |
| 3    | Diseño general y navegación                          | Hecha     |
| 4    | Registro, login y roles                              | Hecha     |
| 5    | Aplicación del cliente y solicitudes                 | Hecha     |
| 6    | Aplicación del prestador y oportunidades             | Hecha     |
| 7    | Cotizaciones, elección y créditos                    | Hecha     |
| 8    | Chat, notificaciones y ejecución del trabajo         | Hecha     |
| 9    | Calificaciones, reclamos y confianza                 | Hecha     |
| 10   | Panel de administración                              | Hecha     |
| 11   | Simulaciones de integraciones externas               | Hecha     |
| 12   | Pruebas integrales y entrega                         | Hecha     |

## Arquitectura propuesta

| Parte | Herramienta | Uso |
| --- | --- | --- |
| Interfaz | React + TypeScript + Vite | App móvil simulada y panel administrativo. |
| Servicio local | Node.js + Express | Reglas, operaciones y API compartida. |
| Base de datos | SQLite | Persistencia local en un solo archivo. |
| Datos iniciales | Script de carga o semilla | Clientes, prestadores, solicitudes, cotizaciones, créditos y trabajos de prueba. |

SQLite permite guardar toda la información en un archivo local y no necesita instalar ni administrar un servidor de base de datos separado. La API local evita que cada modo tenga información aislada.

## Qué significa “100 % funcional”

El prototipo será completamente funcional dentro del alcance que seleccionemos:

- Los formularios validan y guardan información.
- Las acciones modifican la base de datos.
- Cliente, prestador y administración consultan los mismos datos.
- Las reglas de créditos y estados se ejecutan realmente.
- Los errores y casos sin datos tienen respuesta en la interfaz.
- Los datos permanecen después de recargar el navegador.
- Existe una opción para restablecer los datos de demostración.

No significa implementar servicios externos ni preparar el sistema para usuarios reales. SMS, Yape, tarjetas, mapas, push, correos y validación de documentos pueden representarse mediante simuladores locales que produzcan el mismo resultado dentro del flujo.

## Recorrido principal que debe funcionar

1. Ingresar como un cliente de prueba.
2. Publicar una solicitud sin usar créditos.
3. Cambiar a uno o más prestadores aprobados.
4. Consultar la solicitud y enviar cotizaciones sin consumir créditos.
5. Regresar al cliente, comparar y aceptar una cotización.
6. Descontar el crédito únicamente al prestador elegido.
7. Convertir la solicitud en un trabajo asignado.
8. Actualizar el trabajo por los estados acordados.
9. Finalizar y registrar calificaciones mutuas.
10. Revisar usuarios, solicitud, cotizaciones, trabajo y movimientos desde administración.

## Datos de prueba mínimos

- Un cliente con direcciones guardadas e historial.
- Tres prestadores aprobados y uno pendiente de verificación.
- Especialidades, distritos, perfiles, reputación y portafolios.
- Saldos de créditos distintos para probar casos con y sin saldo.
- Solicitudes nuevas, con cotizaciones, asignadas, terminadas y canceladas.
- Cotizaciones elegidas, no elegidas, retiradas y vencidas.
- Conversaciones, notificaciones, movimientos y calificaciones de ejemplo.
- Una cuenta administrativa con acceso a todos los módulos del prototipo.

## Integraciones simuladas

| Integración real futura | Comportamiento del prototipo |
| --- | --- |
| Código por SMS | Mostrar o usar un código fijo de demostración. |
| Compra con Yape o tarjeta | Formulario de pago de prueba que acredita saldo. |
| Mapas y ubicación | Coordenadas de prueba mostradas con Leaflet y cartografía de OpenStreetMap; permite arrastrar, acercar, alejar y seleccionar prestadores. |
| Notificaciones push | Bandeja local y avisos dentro de la interfaz. |
| DNI y selfie | Archivos de prueba y aprobación manual desde administración. |
| Boleta o factura | Comprobante de demostración generado localmente. |

## Límites

- Se ejecutará en una computadora local.
- No manejará información personal real.
- No procesará dinero real.
- No enviará SMS, correos ni notificaciones reales.
- No define todavía la arquitectura del producto de producción.

## Estado de entrega

Las doce fases del prototipo local quedaron implementadas el 2026-10-17. La guía de recorrido se encuentra en `C:\JUBER\FREELANCE\NEEDDOER\docs\DEMO_GUIDE.md`. Se validan tipos, compilación y los recorridos principales mediante API; la revisión visual queda reservada para una sesión posterior.

### Ubicación y distancia de cotizaciones — 2026-10-19

- El perfil del prestador permite guardar un punto de partida habitual por distrito.
- Cada cotización exige elegir el punto habitual o capturar la ubicación actual.
- La ubicación actual reduce su precisión antes de guardarse para no exponer el punto exacto.
- La cotización conserva el punto aproximado y la distancia calculada al lugar del servicio.
- El cliente ve las cotizaciones sobre un mapa local, ordenadas por cercanía, y puede filtrar las del mismo distrito.
- El mapa usa Leaflet y OpenStreetMap, encuadra todos los puntos al abrirse y se acerca al prestador seleccionado.
- La migración `20261019063416_provider_quote_locations` conserva la base existente y el script `npm run db:backfill-locations` completa los datos de demostración anteriores.

### Mapa de oportunidades y cotización separada — 2026-10-20

- El prestador ve en cada oportunidad la zona aproximada del cliente y su propio punto habitual sobre el mapa.
- La API calcula la distancia aproximada entre ambos puntos sin exponer la dirección exacta del cliente.
- El botón **Cotizar** se mantiene visible en la cabecera del detalle.
- Al pulsarlo, el formulario reemplaza el detalle de la oportunidad y una flecha compacta permite regresar.
- El formulario usa campos más compactos y dos columnas cuando hay espacio para evitar desplazamiento vertical innecesario.

### Filtros del prestador — 2026-10-20

- Cotizaciones muestra los grupos Pendientes, Elegidas y Finalizadas con sus cantidades.
- Trabajos muestra En curso y Terminados, con En curso seleccionado inicialmente.
- Cada filtro presenta un mensaje específico cuando no existen registros en ese estado.





