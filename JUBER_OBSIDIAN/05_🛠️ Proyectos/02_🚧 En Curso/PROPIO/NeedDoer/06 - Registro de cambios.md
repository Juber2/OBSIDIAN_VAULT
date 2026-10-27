---
tipo: registro de cambios
actualizado: 2026-10-20
tags:
  - needdoer
  - cambios
---

# Registro de cambios

Volver a [[NeedDoer]].

Aquí se registran cambios que alteran el alcance, la prioridad, el flujo o las reglas del proyecto. Las correcciones de redacción menores no necesitan una entrada.

| Fecha | Cambio | Motivo | Notas afectadas | Responsable |
| --- | --- | --- | --- | --- |
| 2026-10-17 | Se creó el centro de documentación y se importó el mapa funcional inicial. | Preparar una base editable para continuar definiendo el proyecto. | Todas | Juber |
| 2026-10-17 | Se cambió el modelo de créditos: el cliente publica gratis y al prestador solo se le descuenta cuando su cotización es aceptada. | Cobrar al prestador únicamente cuando obtiene el trabajo. | Visión, mapa funcional, reglas, decisiones y roadmap | Juber |
| 2026-10-17 | Se definió inicialmente un prototipo sencillo en React. | Validar la idea y los recorridos antes de tomar decisiones de desarrollo. | Inicio y roadmap | Juber |
| 2026-10-17 | Se amplió el alcance a un prototipo funcional con API local, SQLite, datos de prueba y lógica compartida. | Simular de forma realista la interacción entre cliente, prestador y administración. | Inicio, roadmap y prototipo funcional | Juber |
| 2026-10-17 | Se eligió **NeedDoer** como nombre definitivo del proyecto y se reemplazó el nombre provisional Chambita Ya. | Contar con una marca breve, internacional y relacionada con encontrar a quien realiza el trabajo. | Toda la documentación del proyecto | Juber |
| 2026-10-17 | Se completó la Fase 1 del prototipo en `C:\JUBER\FREELANCE\NEEDDOER`. | Dejar una base ejecutable y comprobada antes de implementar funcionalidades. | Prototipo funcional y roadmap | Codex |
| 2026-10-17 | Se completó la Fase 2 del prototipo: modelo de datos, migraciones y datos de demostración reiniciables. | Contar con una base completa y poblada antes de construir las pantallas. | Prototipo funcional | Claude |
| 2026-10-17 | Se completó la Fase 3: sistema visual, selector de rol, navegación responsive y portadas para cliente, prestador y administración. | Validar la estructura y los recorridos antes de conectar las funciones de negocio. | Prototipo funcional y roadmap | Codex |
| 2026-10-17 | Se completó la Fase 4: registro, login, sesión persistente, cierre de sesión y protección por roles. | Incorporar acceso real al prototipo usando las cuentas y perfiles de la base local. | Prototipo funcional | Codex |
| 2026-10-17 | Se completaron las Fases 5 y 6: gestión de solicitudes del cliente y oportunidades del prestador filtradas por oficio y zona. | Conectar los primeros recorridos operativos con la base local y preparar la cotización. | Prototipo funcional | Codex |
| 2026-10-17 | Se completaron las Fases 7 a 12: cotizaciones y créditos, mensajes y trabajos, confianza, administración, simulaciones y entrega. | Dejar operativo el recorrido integral del prototipo, incluida la regla de cobro solo al aceptar una cotización. | Prototipo funcional, arquitectura y guía de demostración | Codex |
| 2026-10-18 | Se eliminó la selección de perfil de la portada, se separó el acceso administrativo y se añadió el cambio entre cliente y prestador dentro de la sesión. | Mantener una sola app y una sola cuenta para los usuarios, dejando el panel administrativo fuera de ese recorrido. | Visión, mapa funcional, roadmap y prototipo | Juber y Codex |
| 2026-10-18 | Se renombró la sección del cliente como **Servicios contratados** y se habilitaron perfiles públicos desde cada cotización, con experiencia, historial y comentarios. | Evitar lenguaje interno confuso y permitir evaluar al prestador antes de elegirlo. | Mapa funcional y prototipo | Juber y Codex |
| 2026-10-18 | Se tradujeron los estados visibles, se dejó **Abiertas** como filtro inicial y se ampliaron los detalles con descripción, categoría, ubicación, presupuesto, propuesta y fotos de demostración. | Hacer la interfaz comprensible y permitir evaluar correctamente cada solicitud y servicio. | Prototipo funcional | Juber y Codex |
| 2026-10-18 | Se reorganizó Mensajes como una lista de chats al estilo de una aplicación de mensajería, manteniendo una conversación independiente por cotización y servicio. | Evitar mezclar acuerdos de trabajos diferentes y hacer evidente con quién se conversa. | Mapa funcional y prototipo | Juber y Codex |
| 2026-10-18 | Chats muestra prestadores o clientes con servicios activos; el acceso se habilita al aceptar una cotización y se cierra al terminar o cancelar el servicio. | Enfocar la comunicación en servicios contratados y vigentes. | Reglas, mapa funcional y prototipo | Juber y Codex |
| 2026-10-18 | Mis solicitudes conserva solo publicaciones sin prestador contratado; al aceptar una cotización, el caso pasa a Servicios y permanece allí durante todo su ciclo e historial. | Eliminar la duplicación entre solicitudes terminadas y servicios terminados. | Reglas, mapa funcional y prototipo | Juber y Codex |
| 2026-10-18 | Servicios del cliente usa los filtros En curso y Terminadas; los estados operativos activos se presentan como En curso. | Simplificar el seguimiento para el cliente y separar trabajos vigentes del historial. | Mapa funcional y prototipo | Juber y Codex |
| 2026-10-19 | Se hizo obligatorio el punto de partida aproximado al cotizar, usando el punto habitual del perfil o la ubicación actual. El cliente puede ver las propuestas en un mapa, comparar su distancia y filtrar por cercanía o mismo distrito. | Permitir elegir prestadores de la zona sin revelar su dirección exacta. | Reglas, mapa funcional y prototipo | Juber y Codex |
| 2026-10-19 | El botón **Ver cotizaciones** se ubicó en la cabecera y reemplaza temporalmente todo el detalle de la solicitud por las vistas de mapa o lista; en el mapa solo se carga el detalle del prestador seleccionado. | Reducir la saturación visual y evitar una pantalla demasiado larga. | Mapa funcional y prototipo | Juber y Codex |
| 2026-10-19 | La solicitud de demostración **La terma no calienta** pasó de 2 a 5 cotizaciones, con prestadores ubicados entre 0 y 10.6 km. | Hacer visibles las diferencias de distancia y probar mejor el mapa. | Prototipo funcional | Juber y Codex |
| 2026-10-19 | El mapa esquemático de cotizaciones se reemplazó por Leaflet con cartografía de OpenStreetMap, zoom, desplazamiento, encuadre automático, ventanas informativas y marcadores seleccionables. | Permitir una comparación espacial realista y fácil de explorar. | Mapa funcional y prototipo | Juber y Codex |
| 2026-10-19 | Se eliminó la vista paralela de lista y la cotización debajo del mapa. La pantalla muestra solo el mapa; al tocar un marcador lo reemplaza por el detalle elegido y permite regresar. | Aprovechar mejor el espacio móvil y evitar desplazamiento vertical innecesario. | Mapa funcional y prototipo | Juber y Codex |
| 2026-10-19 | Se agregaron seis solicitudes abiertas de demostración para gasfitería, electricidad, termas, lavadoras y refrigeradoras, distribuidas según las especialidades y zonas de los prestadores. | Poblar la sección Oportunidades y permitir probar filtros y cotizaciones. | Prototipo funcional | Juber y Codex |
| 2026-10-20 | El detalle de una oportunidad muestra un mapa con la zona aproximada del cliente, el punto habitual del prestador y la distancia. El botón **Cotizar** aparece arriba y reemplaza el detalle por un formulario independiente. | Ayudar al prestador a decidir si el traslado le conviene y reducir el desplazamiento vertical en móvil. | Reglas, mapa funcional y prototipo | Juber y Codex |
| 2026-10-20 | Se agregaron filtros con contadores en Cotizaciones y Trabajos del prestador. Cotizaciones se divide en Pendientes, Elegidas y Finalizadas; Trabajos en En curso y Terminados. | Facilitar el seguimiento sin mezclar propuestas o servicios en estados distintos. | Mapa funcional y prototipo | Juber y Codex |

## Plantilla

| Fecha | Cambio | Motivo | Notas afectadas | Responsable |
| --- | --- | --- | --- | --- |
| AAAA-MM-DD |  |  |  |  |




