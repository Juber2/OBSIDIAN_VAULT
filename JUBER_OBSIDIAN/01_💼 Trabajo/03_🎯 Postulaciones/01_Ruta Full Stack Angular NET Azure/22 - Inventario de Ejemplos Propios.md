---
tags:
  - Postulaciones
  - Entrevistas
  - Evidencia
Creado: 2026-08-31
Relacionado:
  - 20 - Banco de Preguntas de las 14 Vacantes
  - 13 - Sistema de postulación por vacante
---

# Inventario de ejemplos propios

## El hueco que llena esta nota

En la carpeta de postulaciones hay **91 menciones** a "ejemplo propio", "caso real", "ejemplo concreto" y "evidencia". Cada checklist, cada banco de preguntas y cada nota de vacante repite la misma exigencia.

Pero no existe **ningún lugar donde esos ejemplos estén escritos**. La exigencia está en todas partes; el inventario, en ninguna.

> [!warning] Por qué esto importa más de lo que parece
> Un ejemplo propio no se improvisa bajo presión. En la entrevista, la diferencia entre *"Cache-Aside es un patrón donde primero consultás el caché…"* y *"lo usamos en X para el catálogo, que se leía miles de veces y cambiaba poco; el problema fue la invalidación al actualizar precios"* decide la entrevista. Lo segundo no se puede inventar en el momento: **se prepara antes o no aparece**.

## Punto de partida: lo que tu CV ya prueba

Estas pistas salen de tu CV real (ver [[23 - Mi CV frente a las 14 Vacantes]]), no son inventadas. **Falta el detalle que solo vos tenés**: qué decidiste, qué descartaste y qué resultó.

| Tema | Pista del CV | Qué te falta agregar |
|---|---|---|
| Consulta lenta | *"Optimización de consultas y procedimientos almacenados en SQL Server, reduciendo tiempos de respuesta en módulos críticos"* (Colegio Trener) | ¿Qué módulo? ¿Cómo lo detectaste? ¿Índice, reescritura o procedure? ¿Cuánto bajó? |
| Integración con terceros | *"Integración de pagos mediante QR (Izipay)"* (Pluxee) | ¿Qué pasaba si Izipay no respondía? ¿Reintentos? ¿Cómo evitabas cobrar dos veces? |
| CI/CD y rollback | *"migrando de un esquema manual en IIS a despliegues continuos"* (Pluxee) | ¿Qué se rompía antes? ¿Cómo revertían un deploy fallido? |
| Modernización legacy | *"Migración de módulos legacy a arquitectura moderna de back office"* (Pluxee) | ¿Cómo convivían lo viejo y lo nuevo? ¿Qué migraste primero y por qué? |
| Arquitectura aplicada | *"Clean Architecture, CQRS y MediatR"* (sistema de capacitación) | ¿Por qué CQRS ahí? ¿Qué alternativa descartaste? |
| Alto volumen | *"procesos batch y ETLs para manejo de datos masivos"* | ¿Cuántos registros? ¿Cómo evitabas quedarte sin memoria? |
| Docker | *"Contenerización de aplicaciones con Docker"* (Pluxee) | ¿Qué problema concreto resolvió? |
| Sector financiero | *"proyecto BBVA"* (NTT DATA) + Izipay | Contexto: es tu evidencia de banca, vale para varias vacantes |

> [!warning] Lo que el CV **no** puede cubrir
> No hay nada de **testing, mensajería, seguridad JWT/OAuth, observabilidad ni Kubernetes** en tu experiencia documentada. Esas filas se llenan con el [[06 - Laboratorio Integrado - API de Ordenes]] o quedan vacías — pero no se inventan.

## El resto lo llenás vos

Un ejemplo inventado es peor que ninguno: se cae en la primera repregunta y cuesta la credibilidad de toda la entrevista. Lo que sigue es la estructura y qué temas necesitan un caso sí o sí.

## Formato de cada ejemplo

```
CONTEXTO   → qué proyecto, qué problema de negocio (1 frase)
DECISIÓN   → qué hiciste y por qué elegiste eso
ALTERNATIVA→ qué descartaste y por qué  ← esto es lo que suena a senior
RESULTADO  → qué cambió (métrica si existe, nunca inventada)
APRENDIZAJE→ qué harías distinto hoy
```

La línea de **alternativa descartada** es la que más pesa. Cualquiera cuenta qué hizo; explicar qué no hizo y por qué demuestra criterio.

## De dónde salen tus ejemplos

| Fuente | Qué sacar | Nota |
|---|---|---|
| **Trabajo freelance actual** (StaffTrainingPlatform) | Decisiones de arquitectura, seguridad, base de datos, integraciones que estás tomando ahora | Es la fuente más fresca y la que mejor recordás |
| **Trabajos y proyectos anteriores** | Incidentes, migraciones, refactors, decisiones bajo presión | Requiere sentarse a recordar; hacelo una vez, no cada entrevista |
| **[[06 - Laboratorio Integrado - API de Ordenes]]** | Lo que estás construyendo estas 3 semanas | Cubre los huecos que las otras dos fuentes no tapan |

> El laboratorio existe precisamente para eso: **fabricar evidencia donde no la tenés**. Si al llenar la tabla ves que no tenés ningún caso de mensajería idempotente, ese es el motivo de hacer la fase 5 del laboratorio, no "practicar Service Bus".

## La tabla — llenar durante las 3 semanas

Un ejemplo por tema. Los marcados 🔴 son los de las preguntas que hacen 4+ vacantes: **esos no pueden quedar vacíos**.

### Arquitectura y diseño

| Tema | Mi ejemplo | ☐ |
|---|---|---|
| 🔴 Cuándo NO usar microservicios | | ☐ |
| Arquitectura en capas o hexagonal aplicada | | ☐ |
| Separar dominio de infraestructura | | ☐ |
| Un patrón de diseño que usé y qué resolvía | | ☐ |
| Una decisión técnica con trade-off explícito | | ☐ |

### Datos

| Tema | Mi ejemplo | ☐ |
|---|---|---|
| 🔴 Una consulta lenta que diagnostiqué y corregí | | ☐ |
| Un índice que agregué y cómo verifiqué que servía | | ☐ |
| Un deadlock o problema de concurrencia | | ☐ |
| Elegir SQL vs NoSQL para un caso concreto | | ☐ |
| Una migración de base sin cortar el servicio | | ☐ |

### Integraciones y resiliencia

| Tema | Mi ejemplo | ☐ |
|---|---|---|
| 🔴 Evitar duplicados / idempotencia | | ☐ |
| 🔴 Retry, timeout o circuit breaker en un servicio externo | | ☐ |
| 🔴 Versionar una API sin romper clientes | | ☐ |
| Una integración con un sistema que no controlaba | | ☐ |

### Seguridad

| Tema | Mi ejemplo | ☐ |
|---|---|---|
| 🔴 Manejo de secretos y credenciales | | ☐ |
| Autenticación/autorización que implementé | | ☐ |
| Un problema de permisos que detecté o corregí | | ☐ |

### Entrega y operación

| Tema | Mi ejemplo | ☐ |
|---|---|---|
| 🔴 Un rollback que tuve que hacer | | ☐ |
| 🔴 Correlacionar o diagnosticar entre varios servicios | | ☐ |
| Un pipeline de CI/CD que armé o mejoré | | ☐ |
| Docker en un proyecto real | | ☐ |

### Conductuales — las que caen en casi toda entrevista

| Tema | Mi historia | ☐ |
|---|---|---|
| 🔴 Un error técnico que cometí | | ☐ |
| 🔴 Un incidente difícil (contexto → decisión → resultado) | | ☐ |
| Un desacuerdo de arquitectura y cómo se resolvió | | ☐ |
| Deuda técnica vs entrega con fecha | | ☐ |
| Ayudar a alguien bloqueado / code review | | ☐ |

## Cuándo llenarla

No en un bloque aparte — **mientras estudiás**. Cada vez que en el plan de [[04 - Plan Detallado 3 Semanas (dia a dia)]] aparezca un checklist con "ejemplo propio", ese es el momento de escribirlo acá. Toma 3 minutos y evita tener que reconstruirlo la noche antes de una entrevista.

Los domingos, en la revisión: mirar qué filas 🔴 siguen vacías.

## Qué hacer con los huecos

Una fila vacía no es un fracaso: **es información accionable**. Tres salidas posibles, en este orden:

1. **Buscar más en tu historial.** Muchas veces el ejemplo existe pero no lo asociabas con ese nombre técnico. "Le pusimos un límite de reintentos para no saturar el servicio" *es* un ejemplo de resiliencia, aunque nunca lo llamaste circuit breaker.
2. **Fabricarlo en el laboratorio.** Si no existe en tu historial, la fase correspondiente de [[06 - Laboratorio Integrado - API de Ordenes]] lo genera — y "lo implementé en un proyecto propio, te muestro el repo" es una respuesta perfectamente válida.
3. **Decirlo honestamente.** *"No me tocó en producción; lo implementé en un proyecto propio y sé cómo lo abordaría"* es una respuesta aceptada. Inventar experiencia no lo es, y se detecta en la primera repregunta.

## Uso doble: esto también es tu CV

El mismo material alimenta la fórmula de logros de [[13 - Sistema de postulación por vacante]]:

```
Verbo + problema/alcance + decisión técnica + resultado medible
```

Un ejemplo bien escrito acá se convierte en una línea del CV con muy poca reescritura. Es la razón para invertir los 3 minutos por fila: sirve dos veces.

## Señal de que está lista

- [ ] Todas las filas 🔴 tienen ejemplo — son las de las preguntas más probables.
- [ ] Cada ejemplo incluye la **alternativa que descarté**, no solo lo que hice.
- [ ] Puedo contar cualquiera en **menos de 2 minutos**, en voz alta, sin leer.
- [ ] Ninguna métrica está inventada.
- [ ] Tengo al menos **2 historias conductuales** sólidas (un error, un incidente).
