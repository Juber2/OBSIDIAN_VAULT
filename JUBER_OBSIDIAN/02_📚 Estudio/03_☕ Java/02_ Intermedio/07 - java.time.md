---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
Palabras clave:
  - java.time, LocalDate, LocalDateTime, ZonedDateTime, Duration, Period, DateTimeFormatter
---

## 🧠 Concepto

El paquete `java.time` (Java 8+) reemplaza a `Date` y `Calendar`. Es inmutable, thread-safe y mucho más intuitivo.

> [!important] Evitar usar `java.util.Date` y `Calendar` en código nuevo.

## Clases principales

| Clase | Qué representa |
|-------|---------------|
| `LocalDate` | Solo fecha (año, mes, día) |
| `LocalTime` | Solo hora (hora, minuto, segundo, nano) |
| `LocalDateTime` | Fecha + hora (sin zona horaria) |
| `ZonedDateTime` | Fecha + hora + zona horaria |
| `Instant` | Punto en el tiempo (timestamp UTC) |
| `Duration` | Cantidad de tiempo en horas/minutos/segundos |
| `Period` | Cantidad de tiempo en años/meses/días |

## LocalDate

```java
LocalDate hoy = LocalDate.now();
LocalDate navidad = LocalDate.of(2026, 12, 25);
LocalDate parseado = LocalDate.parse("2026-05-21"); // ISO 8601

// Operaciones
LocalDate manana = hoy.plusDays(1);
LocalDate semanaAntes = hoy.minusWeeks(1);
LocalDate mesProximo = hoy.plusMonths(1);

// Getters
hoy.getYear();        // 2026
hoy.getMonth();       // Month.MAY
hoy.getMonthValue();  // 5
hoy.getDayOfMonth();  // 21
hoy.getDayOfWeek();   // DayOfWeek.THURSDAY

// Comparar
hoy.isBefore(navidad); // true
hoy.isAfter(navidad);  // false
hoy.isEqual(hoy);      // true
```

## LocalTime

```java
LocalTime ahora = LocalTime.now();
LocalTime reunion = LocalTime.of(14, 30, 0); // 14:30:00

ahora.getHour();   // ej: 10
ahora.getMinute(); // ej: 45
ahora.plusHours(2);
ahora.isBefore(reunion);
```

## LocalDateTime

```java
LocalDateTime ahora = LocalDateTime.now();
LocalDateTime fecha = LocalDateTime.of(2026, 5, 21, 10, 30);

// Combinar LocalDate + LocalTime
LocalDate fecha = LocalDate.now();
LocalTime hora = LocalTime.of(9, 0);
LocalDateTime combinado = LocalDateTime.of(fecha, hora);

// Conversión
LocalDate solo = ahora.toLocalDate();
LocalTime horaOnly = ahora.toLocalTime();
```

## ZonedDateTime

```java
ZoneId zona = ZoneId.of("America/Bogota");
ZonedDateTime bogota = ZonedDateTime.now(zona);
ZonedDateTime madrid = bogota.withZoneSameInstant(ZoneId.of("Europe/Madrid"));

// Zonas disponibles
ZoneId.getAvailableZoneIds(); // Set<String>
```

## Instant

```java
Instant ahora = Instant.now(); // timestamp UTC
long epochMilli = ahora.toEpochMilli();

// Convertir a ZonedDateTime
ZonedDateTime zdt = ahora.atZone(ZoneId.systemDefault());
```

## Duration y Period

```java
// Duration — tiempo en segundos/nanosegundos
Duration tresHoras = Duration.ofHours(3);
Duration entre = Duration.between(LocalTime.of(9,0), LocalTime.of(17,0));
entre.toHours();   // 8
entre.toMinutes(); // 480

// Period — tiempo en años/meses/días
Period seisM = Period.ofMonths(6);
Period entre = Period.between(LocalDate.of(2020,1,1), LocalDate.now());
entre.getYears();
entre.getMonths();
entre.getDays();
```

## DateTimeFormatter — formatear y parsear

```java
LocalDateTime ahora = LocalDateTime.now();

// Formatos predefinidos
DateTimeFormatter isoDate = DateTimeFormatter.ISO_LOCAL_DATE;
ahora.format(isoDate); // "2026-05-21"

// Formato custom
DateTimeFormatter formato = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
String texto = ahora.format(formato);    // "21/05/2026 10:30"
LocalDateTime parseado = LocalDateTime.parse("21/05/2026 10:30", formato);

// Localizado
DateTimeFormatter localizado = DateTimeFormatter
    .ofLocalizedDate(FormatStyle.FULL)
    .withLocale(new Locale("es", "CO"));
LocalDate.now().format(localizado); // "jueves, 21 de mayo de 2026"
```

## ⚠️ Errores Comunes

> [!warning] `LocalDateTime` no tiene zona horaria
> Si necesitas manejar múltiples zonas horarias o almacenar en BD con timezone, usa `ZonedDateTime` o `Instant`.

> [!warning] `DateTimeParseException` al parsear
> El formato del string debe coincidir exactamente con el patrón del formatter.

## 📌 Notas Rápidas
- Todas las clases de `java.time` (`LocalDate`, `LocalDateTime`, `Instant`, etc.) son inmutables — cada operación devuelve un objeto nuevo.
- Para timestamps en base de datos/APIs, `Instant` (UTC, sin zona) es la opción más segura para evitar ambigüedades de zona horaria.
- `Duration` mide tiempo entre dos instantes (horas, minutos, segundos); `Period` mide diferencias en fechas de calendario (años, meses, días).
- Reemplaza por completo a `java.util.Date`/`Calendar` (legacy, mutable y con muchos gotchas) — no usar las clases viejas en código nuevo.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

