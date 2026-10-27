---
tipo: reglas de negocio
estado: propuestas por validar
actualizado: 2026-10-20
tags:
  - needdoer
  - reglas
---

# Reglas de negocio

Volver a [[NeedDoer]].

Estas reglas provienen del mapa funcional inicial. Sus valores son propuestas y deben validarse antes de considerarlos definitivos.

| Regla | Propuesta actual | Motivo | Estado |
| --- | --- | --- | --- |
| Costo por cotización aceptada | 1 crédito, con equivalencia referencial de S/1. | El prestador paga únicamente cuando obtiene el trabajo. | Por validar |
| Momento de consumo | Cuando el cliente acepta la cotización. | Enviar o perder una cotización no consume saldo. | Aprobada |
| Saldo para cotizar | El prestador debe tener créditos disponibles al enviar. | Asegurar que una cotización elegida pueda cobrarse; falta decidir si el saldo se reserva. | Parcialmente aprobada |
| Máximo de cotizaciones por solicitud | 5. | Permitir comparación sin reducir demasiado la probabilidad de ganar. | Por validar |
| Vigencia de solicitud urgente | 24 horas si es “lo antes posible”. | Evitar solicitudes abiertas indefinidamente. | Por validar |
| Vigencia de solicitud programada | Hasta 24 horas antes de la fecha elegida. | Cerrar la recepción de ofertas con anticipación. | Por validar |
| Quién puede cotizar | Solo prestadores aprobados. | La verificación sostiene la confianza del servicio. | Propuesta firme |
| Eventos anteriores a la elección | Cancelar, vencer, no ser elegido o retirar una cotización no consume créditos. | No existe un cobro que deba devolverse antes de la aceptación. | Aprobada |
| Devolución después de elegir | Definir cuándo se devuelve el crédito si hay cancelación o reclamo después de la aceptación. | El crédito ya fue consumido al elegir al prestador. | Por decidir |
| Retiro de cotización | Permitido antes de la elección y sin consumo de crédito. | El prestador aún no ha sido elegido. | Aprobada |
| Dirección exacta | Visible solo para el prestador elegido; antes se muestra distrito y zona aproximada. | Proteger la privacidad y seguridad del cliente. | Propuesta firme |
| Ubicación del prestador al cotizar | Toda cotización debe registrar un punto de partida aproximado, tomado del perfil o de la ubicación actual. | Permitir al cliente comparar la cercanía de todas las propuestas. | Definido |
| Precisión del prestador | La ubicación actual se redondea antes de guardarse y el cliente nunca ve la dirección exacta del prestador. | Mostrar una distancia útil sin exponer una vivienda o taller exacto. | Definido |
| Mapa y distancia | El cliente ve los prestadores en un mapa aproximado, la distancia a su solicitud y filtros por cercanía o mismo distrito. | Facilitar que elija a alguien de su zona cuando lo prefiera. | Definido |
| Mapa para el prestador | Antes de cotizar, el prestador ve la zona aproximada del servicio, su punto habitual y la distancia entre ambos. | Permitir que decida si el desplazamiento le conviene sin revelar la dirección exacta del cliente. | Definido |
| Apertura del chat | Se habilita cuando el cliente acepta una cotización y permanece disponible mientras el servicio está activo. | La coordinación directa empieza cuando ya existe un acuerdo entre cliente y prestador. | Definido |
| Cierre del chat | Deja de mostrarse y no permite nuevos mensajes cuando el servicio termina o se cancela. | Evitar conversaciones abiertas sin un servicio vigente; los mensajes se conservan en la base de datos. | Definido |
| Separación entre solicitudes y servicios | La solicitud permanece en Mis solicitudes hasta elegir un prestador. Después se muestra solo en Servicios, incluso al terminar o cancelar el trabajo. | Evitar que el mismo caso aparezca duplicado en ambas secciones. | Definido |
| Créditos de bienvenida | 10 al aprobar al prestador. | Permitir que pruebe el servicio antes de comprar. | Por validar |
| Confirmación automática | Finalizar 48 horas después de que el prestador marque el trabajo, si el cliente no responde. | Cerrar el ciclo y habilitar la calificación. | Por validar |
| Cancelación del cliente | Sin penalidad antes de “en camino”; después queda registrada. | Proteger el tiempo del prestador. | Por validar |
| Cancelación del prestador | 3 cancelaciones en 30 días activan revisión de cuenta. | Proteger la confianza del cliente. | Por validar |
| Calificación baja | Promedio menor a 3.5 con 10 trabajos o más activa revisión. | Mantener la calidad de la oferta. | Por validar |

## Reglas de créditos

- El saldo siempre es visible en el modo prestador.
- El cliente no compra ni usa créditos para publicar solicitudes.
- Antes de enviar una cotización se muestra su costo potencial y el saldo disponible del prestador.
- Enviar una cotización no modifica el saldo.
- El crédito se descuenta únicamente al aceptar el cliente la cotización.
- Las cotizaciones no elegidas, retiradas o vencidas no consumen créditos.
- Los movimientos distinguen compras, consumos por cotizaciones aceptadas, reembolsos y bonos.
- La política de devolución después de una aceptación queda pendiente.

## Reglas de privacidad y confianza

- Solo los prestadores aprobados pueden cotizar.
- El equipo revisa manualmente el DNI y la selfie.
- El cliente ve el perfil, la reputación y los sellos antes de elegir.
- La dirección exacta se mantiene oculta hasta la elección.
- Antes de cotizar, el prestador solo ve el punto aproximado de la zona del cliente y la distancia desde su punto habitual.
- El punto de partida del prestador es obligatorio en cada cotización y se conserva como una captura de ese momento.
- El cliente ve el marcador aproximado y la distancia, pero no las coordenadas exactas ni la dirección del prestador.
- Las cancelaciones posteriores a la elección quedan registradas en la reputación de quien cancela.
- Las calificaciones se publican cuando ambos califican o al cumplirse siete días.

## Control de cambios

Cada regla aprobada debe pasar de **Por validar** o **Por decidir** a **Aprobada**, con fecha y responsable en [[06 - Registro de cambios]].

