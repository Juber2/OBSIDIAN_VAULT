---
tags:
  - Java
  - System Design
  - Entrevistas
Creado: 2026-09-08
Relacionado:
  - System Design
  - 01 - Metodo de Diseno
Palabras clave:
  - estimacion, QPS, capacity planning, latencia, almacenamiento, ancho de banda
---

## 🧠 Concepto

Estimar no es adivinar: es **acotar el orden de magnitud** para que el diseño sea proporcional al problema.

La diferencia entre 100 peticiones por segundo y 100.000 no es de grado, es de arquitectura: la primera la resuelve una instancia con Postgres; la segunda necesita caché, particionado y probablemente otra base.

> [!tip] Nadie espera precisión, se espera criterio
> Si estimás 23 QPS y en realidad son 31, da igual: la conclusión de diseño es la misma. Lo que se evalúa es que **sepas qué números importan** y que llegues a una conclusión con ellos.

## 🔢 Los números de memoria

### Potencias de 2

| Potencia | Valor aproximado | Nombre |
|---|---|---|
| 2¹⁰ | 1.000 | mil (KB) |
| 2²⁰ | 1 millón | millón (MB) |
| 2³⁰ | 1.000 millones | mil millones (GB) |
| 2⁴⁰ | 1 billón | billón (TB) |

### Tiempo

```
1 día      = 86.400 s  ≈  100.000 s      ← redondear a 10⁵ simplifica todo
1 mes      ≈ 2,5 millones de s
1 año      ≈ 31,5 millones de s  ≈  3 × 10⁷
```

> [!tip] El truco que más acelera las cuentas
> Redondear un día a **100.000 segundos**. Un millón de eventos por día son 10 por segundo. Diez millones son 100 por segundo. La aritmética sale de cabeza.

### Latencias

| Operación | Tiempo |
|---|---|
| Caché L1 | 1 ns |
| Acceso a RAM | 100 ns |
| Compresión de 1 KB | 10 µs |
| SSD lectura aleatoria | 150 µs |
| Red en el mismo datacenter | 500 µs |
| Lectura secuencial de 1 MB (SSD) | 1 ms |
| Disco mecánico, búsqueda | 10 ms |
| Red intercontinental | 150 ms |

> [!danger] La latencia de red tiene un piso físico
> Lima ↔ Frankfurt son ~10.000 km. A la velocidad de la luz en fibra (~200.000 km/s) la ida y vuelta es ~100 ms **como mínimo teórico**. Ninguna optimización baja de ahí: si el SLA exige menos, necesitás presencia local, no mejor código.

### Tamaños típicos

| Dato | Tamaño |
|---|---|
| Carácter ASCII | 1 byte |
| UUID | 16 bytes (36 como texto) |
| Timestamp | 8 bytes |
| Fila típica de base | 100-500 bytes |
| Registro de log en JSON | 300 B - 2 KB |
| Respuesta JSON pequeña | 1-5 KB |
| Imagen comprimida | 200 KB - 2 MB |

## 📈 Estimar QPS

```
Peticiones por día → QPS promedio → QPS pico

QPS promedio = peticiones_por_dia / 100.000
QPS pico     = QPS promedio × factor
```

| Tipo de sistema | Factor de pico |
|---|---|
| Uso constante (API interna) | ×2 |
| Uso humano diurno | ×3 a ×5 |
| Muy estacional (comercio, banca fin de mes) | ×10 a ×20 |

```
Ejemplo — banca:
  10 M usuarios, 20% activos por día, 3 operaciones cada uno
  = 2 M peticiones/día
  = 2M / 100.000 = 20 QPS promedio
  × 5 (pico horario) = 100 QPS pico
  × 3 (fin de mes, día de pago) = 300 QPS en el peor caso
```

> [!warning] Diseñá para el pico, no para el promedio
> El sistema no se cae en el promedio: se cae en el pico. Y en banca los picos son predecibles: día de pago, fin de mes, fecha de vencimiento de impuestos. Preguntá por ellos en el paso 1.

## 💾 Estimar almacenamiento

```
Almacenamiento = registros/día × tamaño × días de retención × factor de índices
```

```
Ejemplo:
  2 M movimientos/día × 200 bytes           = 400 MB/día
  × 365                                     = 146 GB/año
  × 10 años (retención normativa)           = 1,5 TB
  × 1,5 (índices y sobrecarga)              ≈ 2,2 TB
  × 3 (réplicas)                            ≈ 6,6 TB
```

> [!tip] Multiplicá por índices y por réplicas
> Los índices ocupan entre el 20% y el 100% del tamaño de los datos. Y lo que guardás se replica: 3 copias es lo normal. Olvidar esos factores subestima el almacenamiento por 4 o 5 veces.

## 📡 Estimar ancho de banda

```
Ancho de banda = QPS × tamaño de respuesta
```

```
  100 QPS × 5 KB = 500 KB/s ≈ 4 Mbps       (trivial)
10.000 QPS × 5 KB = 50 MB/s ≈ 400 Mbps     (significativo)
10.000 QPS × 500 KB = 5 GB/s               (necesitás CDN)
```

> [!danger] El ancho de banda saliente se factura
> 50 MB/s sostenidos son ~130 TB al mes de egreso. Es un costo que rara vez aparece en la estimación inicial y que crece exactamente con el éxito — ver [[13 - Costos y FinOps]].
> Mencionarlo en una entrevista de diseño demuestra que pensás el sistema completo, no solo el diagrama.

## 🖥️ Estimar servidores

```
Servidores = QPS pico / (QPS por servidor) × factor de holgura
```

| Tipo de carga | QPS por instancia (referencia) |
|---|---|
| Servir contenido en caché | 5.000 - 20.000 |
| API simple con lectura de base | 500 - 2.000 |
| API con lógica y varias consultas | 100 - 500 |
| Procesamiento intensivo | 10 - 100 |

```
  300 QPS pico / 300 QPS por instancia = 1 instancia
  × 2 (holgura) = 2
  × 2 (dos zonas) = 4 instancias
```

> [!warning] Los números por instancia son referencias, no verdades
> Dependen del hardware, del framework y de qué hace cada petición. Lo importante es el **orden de magnitud**: sirve para saber si necesitás 4 instancias o 400, que es una decisión de arquitectura distinta.

## 🧮 Un caso completo

**Sistema de transferencias, banco mediano.**

```
1 · USUARIOS
    5 M clientes, 30% activos por día = 1,5 M usuarios/día

2 · OPERACIONES
    Escrituras: 1,5 M × 0,5 transferencias  = 750 K/día
    Lecturas:   1,5 M × 8 consultas          = 12 M/día
    → proporción 16:1, sistema de LECTURA intensiva

3 · QPS
    Escrituras: 750 K / 100 K   =   7,5 QPS promedio → ~40 pico
    Lecturas:    12 M / 100 K   =   120 QPS promedio → ~600 pico

4 · ALMACENAMIENTO
    750 K movimientos/día × 250 B = 190 MB/día
    × 365 × 10 años               = 685 GB
    × 1,5 índices × 3 réplicas    ≈ 3 TB

5 · ANCHO DE BANDA
    600 QPS × 3 KB = 1,8 MB/s ≈ 15 Mbps    (bajo)

6 · CACHÉ
    Saldos de usuarios activos: 1,5 M × 100 B = 150 MB
    → entra holgado en memoria ✅
```

**Conclusiones de diseño** — que es el único punto de haber estimado:

| Número | Consecuencia |
|---|---|
| 40 QPS de escritura | una instancia de Postgres sobra; **no hace falta shardear** |
| 16:1 lectura/escritura | réplicas de lectura + caché son la palanca principal |
| Caché de 150 MB | todos los saldos activos caben en memoria |
| 3 TB en 10 años | una instancia grande aguanta; particionado por fecha para el archivado |
| Egreso bajo | no es un factor |

> [!tip] Terminá siempre con las conclusiones
> Los números por sí solos no valen nada. Lo que se evalúa es la frase *"con 40 QPS de escritura no necesito shardear, así que descarto esa complejidad"*. **Estimás para poder descartar.**

## 🚫 Errores al estimar

> [!warning] Los que se ven seguido
> - **No estimar** y diseñar para "escala web" un sistema de 100 QPS.
> - **Solo el promedio**, ignorando el pico.
> - **Olvidar índices y réplicas** en el almacenamiento.
> - **Ignorar el egreso** de datos.
> - **Precisión falsa**: perder tres minutos en 86.400 vs 100.000.
> - **No sacar conclusiones**: estimar y no usar los números.
> - **No preguntar por la retención**: en banca son 10 años, no 90 días.

## 🎯 Guía rápida

```
1 · Usuarios activos por día
2 · Operaciones por usuario → total por día
3 · Dividir por 100.000 → QPS promedio
4 · × factor de pico → QPS pico
5 · Tamaño × volumen × retención × 1,5 × 3 → almacenamiento
6 · QPS × tamaño de respuesta → ancho de banda
7 · ¿Qué entra en memoria? → estrategia de caché
8 · CONCLUSIONES: qué necesito y qué puedo descartar
```

## 📌 Notas Rápidas
- El paso 8 es el que importa. Los siete anteriores son aritmética; el octavo es criterio de ingeniería.
- Redondear un día a 100.000 segundos es el atajo que más simplifica las cuentas mentales. Vale internalizarlo.
- En banca hay dos números que casi nadie pregunta y cambian el diseño: la **retención normativa** (10 años, no 90 días) y los **picos estacionales** (día de pago, fin de mes).
- La proporción lectura/escritura es la primera conclusión que hay que decir en voz alta: orienta toda la arquitectura.

## 🔗 Relaciones
- **Módulo**: [[System Design]]
- **Anterior**: [[01 - Metodo de Diseno]]
- **Siguiente**: [[03 - APIs a Escala]]
- **Se conecta con**: [[13 - Costos y FinOps]], [[Performance Java]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

