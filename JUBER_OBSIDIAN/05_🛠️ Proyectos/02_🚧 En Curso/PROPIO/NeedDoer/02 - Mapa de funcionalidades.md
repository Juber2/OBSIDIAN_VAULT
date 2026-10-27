# NeedDoer — Mapa de funcionalidades

Oct 17, 2026 · @Juber

## Alcance y actores

NeedDoer conecta a quien tiene un problema en casa con técnicos verificados que compiten con cotizaciones. El cliente publica solicitudes gratis. El prestador necesita créditos disponibles para cotizar, pero el crédito solo se consume cuando el cliente acepta su cotización.

| Pieza | Para quién | Plataforma |
| --- | --- | --- |
| App, modo cliente | Quien necesita una reparación | Android e iOS |
| App, modo prestador | Gasfiteros, electricistas, técnicos de electrodomésticos, cerrajeros | La misma app; se activa desde una cuenta iniciada |
| Panel de administración | Equipo de NeedDoer | Web |

Categorías iniciales: gasfitería, electricidad, electrodomésticos y cerrajería.

Prioridad en las tablas: **MVP** = necesario para el primer lanzamiento; **Fase 2** = siguiente versión; **Por decidir** = abierto. Puedes cambiarla en cada fila.

### Ciclo principal

&#91;embedded content: ciclo de una solicitud · 5 estados, 3 salidas\]

Antes de que el cliente elija no se consume ningún crédito. Cuando acepta una cotización, se descuenta el crédito al prestador elegido. Una cancelación después de elegir queda registrada en la reputación de quien cancela; la devolución del crédito en ese caso queda por definir.

## Funciones comunes

Todo usuario entra con su celular y empieza como cliente; el modo prestador se activa aparte. Estas funciones las comparten ambos modos.

### Acceso y cuenta

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Bienvenida | Tres pantallas que explican cómo funciona; se pueden saltar | MVP |
| Registro con celular | Número + código por SMS, sin contraseña | MVP |
| Ingreso con Google o Apple | Alternativa al SMS. Si se ofrece Google en iOS, Apple exige también una opción equivalente como Sign in with Apple | Fase 2 |
| Datos básicos | Nombre, foto y correo opcionales | MVP |
| Consentimiento de datos | Términos, política de privacidad y consentimiento expreso para ubicación, fotos y DNI (Ley 29733) | MVP |
| Direcciones guardadas | Casa, trabajo u otras, con pin en el mapa y referencia | MVP |
| Cambio de modo | Interruptor cliente ↔ prestador en el menú | MVP |
| Eliminar cuenta | Desde la app, con sus datos; lo exigen Google Play y App Store | MVP |

### Chat

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Chat por cotización | Se abre cuando el prestador cotiza; un hilo por solicitud y prestador | MVP |
| Texto y fotos | Mensajes y fotos del problema | MVP |
| Estado del mensaje | Enviado y leído | MVP |
| Reportar y bloquear | Desde el mismo chat | MVP |
| Cierre del chat | Deja de mostrarse y bloquea nuevos mensajes al terminar o cancelar el trabajo | MVP |
| Respuestas rápidas | Frases predefinidas ("¿Qué marca es?", "Estoy en camino") | Fase 2 |
| Notas de voz | Audios cortos | Fase 2 |

### Notificaciones

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Push | Aviso de cada evento; el detalle está en cada módulo | MVP |
| Bandeja | Historial de notificaciones dentro de la app | MVP |
| Preferencias | Elegir qué avisos recibir | Fase 2 |

### Calificaciones

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Calificación mutua | 1 a 5 estrellas, etiquetas ("puntual", "dejó limpio") y comentario al finalizar | MVP |
| Publicación a ciegas | Se publican cuando ambos califican o a los 7 días, para evitar represalias | MVP |
| Respuesta pública | El calificado puede responder una reseña | Fase 2 |

### Soporte y ajustes

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Centro de ayuda | Preguntas frecuentes | MVP |
| Contactar soporte | Formulario dentro de la app | MVP |
| Ajustes | Permisos de ubicación y notificaciones, términos, versión | MVP |

## Cliente

El cliente usa la app gratis: publica su problema, compara cotizaciones, elige y califica.

### Inicio

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Pantalla de inicio | Botón principal "¿Qué necesitas arreglar?", categorías y solicitudes activas con su estado | MVP |
| Buscador | Buscar por palabra ("terma", "chapa") y llegar a la subcategoría | Fase 2 |

### Publicar solicitud (paso a paso)

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Categoría y subcategoría | Ej.: Electrodomésticos › Lavadora › No centrifuga | MVP |
| Descripción | Texto libre del problema | MVP |
| Preguntas guiadas | Preguntas por subcategoría: marca, modelo, antigüedad | Fase 2 |
| Fotos | Hasta 5 fotos del problema | MVP |
| Video corto | Video del ruido o la falla | Fase 2 |
| Ubicación | Dirección guardada o pin; los prestadores solo ven el distrito y una zona aproximada | MVP |
| Cuándo | "Lo antes posible", hoy o una fecha con franja horaria | MVP |
| Presupuesto referencial | Monto opcional; orienta pero no obliga | MVP |
| Resumen y publicar | Revisión final antes de publicar | MVP |

### Mis solicitudes

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Lista por estado | Abiertas, canceladas antes de contratar y vencidas | MVP |
| Detalle | Estado, cotizaciones recibidas y tiempo restante | MVP |
| Editar | Solo mientras no tenga cotizaciones | MVP |
| Cancelar | Con motivo; antes de elegir no hay créditos que devolver porque las cotizaciones aún no generan consumo | MVP |
| Sin cotizaciones | Aviso y opción de ampliar la zona o republicar | MVP |
| Paso a servicios | Al aceptar una cotización, la solicitud deja esta sección y se administra desde Servicios | MVP |

### Comparar y elegir

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Lista de cotizaciones | Monto, tipo (precio cerrado o visita de diagnóstico), calificación, trabajos completados, distancia aproximada, disponibilidad y sello de verificado | MVP |
| Mapa de cotizaciones | Mapa interactivo con calles, arrastre, zoom, ajuste automático y marcadores seleccionables para el servicio y los prestadores; no revela la dirección exacta del prestador | MVP |
| Vista de cotizaciones | El botón aparece en la cabecera y reemplaza el detalle por un mapa amplio. Al tocar un marcador, el mapa se reemplaza por esa cotización y un botón compacto permite regresar | MVP |
| Ordenar y filtrar | Por cercanía, mismo distrito o todas las cotizaciones | MVP |
| Ver perfil del prestador | Perfil público completo | MVP |
| Contactar | Habilita el chat únicamente después de aceptar la cotización y mientras el servicio esté activo | MVP |
| Elegir | Confirma al elegido, avisa a los demás y revela la dirección exacta solo al elegido | MVP |
| Descartar cotización | Ocultar una cotización que no interesa | Fase 2 |

### Trabajo en curso

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Filtros del cliente | En curso agrupa pendiente, confirmado, en camino y trabajando; Terminadas reúne completados y cancelados | MVP |
| Seguimiento por estados | Asignado → en camino → llegó → en curso → finalizado | MVP |
| Datos del prestador | Foto, nombre y sello de verificado para reconocerlo en la puerta | MVP |
| Confirmar finalización | El cliente confirma; si no responde, se da por terminado tras un plazo | MVP |
| Reportar problema | No llegó, cobró distinto a lo cotizado o mal trabajo | MVP |
| Compartir el trabajo | Enviar a un familiar quién viene y a qué hora | Fase 2 |

Los servicios activos, terminados y cancelados después de contratar se muestran únicamente en **Servicios**, para evitar duplicarlos en **Mis solicitudes**.

### Después del trabajo

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Calificar al prestador | Ver calificación mutua | MVP |
| Historial | Trabajos pasados con prestador y monto | MVP |
| Favoritos | Guardar prestadores | Fase 2 |
| Pedir directo a un favorito | Solicitud que solo ve ese prestador | Fase 2 |
| Volver a pedir | Repetir una solicitud anterior | Fase 2 |

## Prestador

El prestador se registra, pasa una verificación manual y necesita créditos disponibles para cotizar. Enviar una cotización no consume créditos: el descuento ocurre únicamente si el cliente la acepta. Su perfil profesional es la parte "Computrabajo" de la app.

### Alta y verificación

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Activar modo prestador | Desde el menú del cliente | MVP |
| Oficios y especialidades | Categorías y subcategorías que atiende | MVP |
| Experiencia | Años en el oficio y descripción breve | MVP |
| Zona de trabajo | Distritos que cubre | MVP |
| Identidad | Foto del DNI (frente y reverso) + selfie, revisadas por el admin | MVP |
| Antecedentes | Subir CertiAdulto o CertiJoven en PDF; da un sello extra | Por decidir |
| Estado de revisión | Pendiente, aprobado u observado con motivo y opción de corregir | MVP |
| Comprobantes | Indicar si emite boleta o recibo por honorarios | Fase 2 |

### Perfil profesional

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Perfil público | Foto, nombre, oficios, presentación, distritos, sellos, calificación, reseñas, trabajos completados y fecha de ingreso | MVP |
| Punto de partida habitual | El prestador guarda un distrito o zona aproximada para calcular distancias al cotizar | MVP |
| Portafolio | Fotos de trabajos hechos (antes y después) con descripción | MVP |
| Vista previa | Ver su perfil como lo ve el cliente | MVP |
| Certificados y estudios | Títulos técnicos y cursos | Fase 2 |
| Métricas públicas | Tasa y tiempo promedio de respuesta | Fase 2 |

### Disponibilidad y solicitudes

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Interruptor "Disponible" | Solo recibe avisos de solicitudes urgentes cuando está activo | MVP |
| Lista de solicitudes | Solicitudes de sus oficios y su zona, las más recientes primero | MVP |
| Filtros | Por categoría, distancia y urgencia | MVP |
| Detalle de solicitud | Descripción, fotos, zona aproximada, cuándo, presupuesto, cupos restantes y distancia desde el punto habitual del prestador | MVP |
| Ocultar solicitud | "No me interesa" | MVP |
| Horario | Días y horas en que trabaja | Fase 2 |
| Mapa de la oportunidad | Muestra la zona aproximada del cliente y el punto habitual del prestador, con distancia, calles, arrastre y zoom | MVP |

### Cotizar

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Enviar cotización | Tipo, monto, qué incluye, mensaje, disponibilidad y punto de partida aproximado obligatorio | MVP |
| Vista independiente para cotizar | El botón Cotizar aparece en la cabecera del detalle y lo reemplaza por el formulario; una flecha compacta permite regresar | MVP |
| Origen de la cotización | Usa el punto habitual del perfil o la ubicación actual; la ubicación actual reduce su precisión antes de compartirse | MVP |
| Validar crédito | Muestra el costo y el saldo antes de enviar; aclara que el crédito solo se consumirá si el cliente acepta la cotización | MVP |
| Mis cotizaciones | Filtros con contadores para pendientes, elegidas y finalizadas; muestra estados enviados, elegidos, no elegidos, retirados y vencidos | MVP |
| Retirar cotización | Antes de que el cliente elija; no consume crédito | MVP |
| Plantillas | Mensajes guardados para cotizar rápido | Fase 2 |

### Trabajos

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Mis trabajos | Agenda de trabajos asignados con filtros En curso y Terminados, seleccionado En curso por defecto | MVP |
| Cómo llegar | Dirección exacta + abrir en Google Maps o Waze | MVP |
| Actualizar estado | En camino, llegué, iniciado, finalizado | MVP |
| Monto final | Registrar cuánto cobró; sirve de respaldo en reclamos | MVP |
| Cancelar | Con motivo; cuenta en su reputación | MVP |
| Calificar al cliente | Ver calificación mutua | MVP |

### Créditos

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Saldo | Visible siempre en el modo prestador | MVP |
| Comprar paquetes | Paquetes de créditos con Yape o tarjeta | MVP |
| Movimientos | Compras, consumos por cotizaciones aceptadas, reembolsos y bonos | MVP |
| Comprobante electrónico | Boleta o factura por cada compra de créditos | MVP |
| Créditos de bienvenida | Se dan al ser aprobado | MVP |
| Incentivo por cotizaciones no elegidas | Revisar la idea original de dar 1 crédito tras perder 3 seguidas; ahora las cotizaciones no elegidas no consumen saldo | Por decidir |
| Referidos | Créditos por invitar a colegas que sean aprobados | Fase 2 |

### Rendimiento

| Funcionalidad | Qué hace | Prioridad |
| --- | --- | --- |
| Mis estadísticas | Cotizaciones enviadas, % elegidas, montos registrados y calificación | Fase 2 |

## Panel de administración (web)

El panel sirve para operar: aprobar prestadores, resolver reclamos y ajustar créditos y categorías.

| Módulo | Qué hace | Prioridad |
| --- | --- | --- |
| Dashboard | Solicitudes, cotizaciones, % con al menos una cotización, % contratadas, tiempo a la primera cotización y ventas de créditos | MVP |
| Verificación | Cola de prestadores pendientes; ver documentos; aprobar, observar con motivo o rechazar | MVP |
| Usuarios | Buscar, ver detalle e historial, suspender o banear | MVP |
| Solicitudes y trabajos | Ver cualquier solicitud con sus cotizaciones y chat; cancelar | MVP |
| Reportes y reclamos | Bandeja de reportes; resolver y dejar nota | MVP |
| Catálogo | Categorías, subcategorías e íconos | MVP |
| Créditos | Paquetes y precios, consumos por cotizaciones aceptadas, ajustes manuales y reembolsos | MVP |
| Moderación de reseñas | Ocultar reseñas que incumplen las normas | Fase 2 |
| Promociones | Cupones y bonos | Fase 2 |
| Reglas configurables | Editar costo por cotización aceptada, cupos y plazos sin publicar una versión nueva | Fase 2 |
| Contenido | Preguntas frecuentes, términos y avisos masivos | Fase 2 |
| Roles del equipo | Admin, soporte y verificador con permisos distintos | Fase 2 |
| Auditoría | Registro de quién hizo qué en el panel | Fase 2 |

## Reglas de negocio

Estas reglas deciden qué muestran varias pantallas; los valores son propuestas para discutir.

| Regla | Propuesta | Motivo |
| --- | --- | --- |
| Costo por cotización aceptada | 1 crédito = S/1 | Valor referencial de la tesis; el prestador paga únicamente cuando obtiene el trabajo |
| Momento de consumo | Al aceptar el cliente la cotización | Enviar, perder, retirar o dejar vencer una cotización no consume créditos |
| Saldo para cotizar | El prestador debe contar con créditos disponibles al enviar | Evita cotizaciones que no podrían cobrarse si resultan elegidas; falta definir si el saldo se reserva |
| Máximo de cotizaciones por solicitud | 5 | Suficiente para comparar sin saturar al cliente ni reducir demasiado la probabilidad de ser elegido |
| Vigencia de la solicitud | 24 h si es "lo antes posible"; hasta 24 h antes de la fecha si es programada | Evita solicitudes abiertas para siempre |
| Quién puede cotizar | Solo prestadores aprobados | La verificación es el diferenciador de confianza |
| Antes de la elección | No hay consumo ni devolución: cancelar, vencer, no ser elegido o retirar la cotización no modifica el saldo | El crédito solo se consume cuando el cliente acepta una cotización |
| Devolución después de elegir | Por definir para cancelaciones y reclamos posteriores a la aceptación | El crédito ya fue consumido y se necesita una política de reverso |
| Retiro de cotización | Permitido antes de la elección y sin consumo de crédito | El prestador puede retirarse mientras el cliente no haya aceptado |
| Dirección exacta | Solo distrito y zona aproximada hasta que el cliente elige | Privacidad y seguridad del cliente |
| Apertura del chat | Al aceptar la cotización; disponible mientras el servicio esté activo | Cliente y prestador coordinan únicamente después de concretar el servicio |
| Cierre del chat | Al terminar o cancelar el servicio; deja de mostrarse y bloquea nuevos mensajes | Mantiene la lista enfocada en coordinaciones vigentes |
| Créditos de bienvenida | 10 créditos al ser aprobado | Permite probar sin pagar |
| Confirmación automática | El trabajo se da por terminado 48 h después de que el prestador lo marca, si el cliente no responde | Cierra el ciclo y habilita la calificación |
| Cancelación del cliente | Sin penalidad antes de "en camino"; después queda registrada | Protege el tiempo del prestador |
| Cancelación del prestador | 3 cancelaciones en 30 días → revisión de la cuenta | Protege la confianza del cliente |
| Calificación baja | Promedio menor a 3.5 con 10 trabajos o más → revisión | Mantiene la calidad de la oferta |

## Requisitos transversales

Estos requisitos aplican a todas las pantallas.

- **Datos personales:** DNI y selfie son datos sensibles según la Ley 29733. Requieren consentimiento expreso, acceso restringido y un plazo de conservación definido.
- **Ubicación:** la dirección exacta nunca se muestra antes de la elección.
- **Celulares modestos:** la app debe funcionar bien en gama media o baja y con datos móviles. Las fotos se comprimen antes de subirlas.
- **Lenguaje simple:** textos grandes, buen contraste y palabras del día a día. Muchos técnicos usan pocas apps.
- **Notificaciones confiables:** si el push no llega, el ciclo se cae.

## Fuera de alcance

Estas funciones quedan fuera a propósito:

- Cobrar el servicio dentro de la app (pago retenido).
- Seguimiento GPS del prestador en tiempo real.
- Empleos formales con postulaciones.
- Cuentas de empresa.
- Verificación automática de identidad y antecedentes.
- Contraoferta del cliente.
- Versión web para clientes.

