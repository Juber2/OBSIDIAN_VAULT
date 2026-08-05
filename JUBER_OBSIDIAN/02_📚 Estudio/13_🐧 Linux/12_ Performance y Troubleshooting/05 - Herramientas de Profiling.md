---
tags:
  - Linux
  - Performance
  - Linux-Performance
Creado: 2026-07-28
Relacionado:
  - Performance y Troubleshooting
  - LINUX
---

# 05 - Herramientas de Profiling

## Que vas a aprender

En este capitulo vas a cerrar el libro con herramientas de **profiling** de nivel senior: `strace` (ya visto de forma introductoria) en profundidad para diagnostico de syscalls, `perf` para profiling de CPU a bajo nivel, y como decidir cuando escalar de monitoreo basico a profiling profundo.

## Explicacion

`strace` (ya visto en el capitulo de Fundamentos) rastrea cada **syscall** que un proceso realiza, extremadamente util para diagnosticar por que un proceso esta lento o colgado a nivel de interaccion con el kernel: archivos que intenta abrir y falla, conexiones de red que tardan, o bloqueos esperando un recurso. `strace -c` da un resumen agregado de tiempo pasado en cada tipo de syscall, identificando rapidamente donde se concentra el tiempo.

`perf` (Linux perf_events) es una herramienta de profiling de CPU mucho mas profunda, capaz de mostrar exactamente en que **funciones** especificas del codigo (incluso a nivel de instruccion de CPU) un proceso pasa su tiempo, usando muestreo estadistico de bajo overhead. Esto conecta directamente con el capitulo de Profiling del libro de NestJS (Clinic.js): mientras Clinic.js es especifico para Node.js y de mas alto nivel, `perf` funciona con cualquier proceso del sistema, dando una vista de profiling universal a nivel de sistema operativo.

## Modelo mental

Pensa `strace` como seguir a un empleado con un cronometro anotando cada pequeña interaccion que tiene con recursos compartidos del edificio (archivos, red, permisos), identificando donde pierde tiempo esperando. `perf` es un analisis mucho mas profundo, casi como un electroencefalograma que muestra exactamente que "musculos mentales" especificos (funciones de codigo) esta usando mas ese empleado durante su trabajo, con precision de nivel de instruccion.

## Ejemplo base

```bash
strace -c comando                 # resumen de tiempo por tipo de syscall
strace -p 1234                     # adjunta a un proceso ya corriendo por PID
strace -T comando                  # muestra tiempo de cada syscall individual

sudo perf record -p 1234 -g -- sleep 30   # graba profiling de CPU de un proceso, 30 segundos
sudo perf report                            # analiza el resultado, mostrando funciones "calientes"

sudo perf top                                # vista en vivo de que funciones consumen mas CPU ahora
```

## Como se usa en un proyecto real

1. Se usa `strace` cuando se sospecha que un proceso esta lento por interacciones especificas con el kernel (archivos, red, permisos), no por logica interna de computo.
2. Se usa `perf` cuando el problema parece ser de computo puro dentro del codigo de la aplicacion, identificando exactamente que funciones consumen mas CPU.
3. Se escala de monitoreo basico (`top`, `iostat`, `vmstat`) a profiling profundo (`strace`, `perf`) solo cuando las herramientas de mas alto nivel no dan suficiente informacion para resolver el problema.
4. Se conecta esto con el capitulo de Profiling del libro de NestJS: `perf` da una vista de sistema operativo que complementa el profiling especifico de Node.js con Clinic.js.
5. Se documenta cada sesion de profiling profundo como caso de estudio, dado que estas herramientas requieren experiencia acumulada para interpretar correctamente sus resultados.

## Criterio junior

Un junior debe poder ejecutar `strace -c` sobre un comando simple e interpretar que tipo de syscalls consume mas tiempo.

## Criterio mid

Un mid debe adjuntar `strace` a un proceso ya corriendo para diagnosticar un problema en tiempo real, distinguiendo cuando el problema es de interaccion con el kernel versus computo puro.

## Criterio senior

Un senior debe usar `perf` para profiling de CPU profundo cuando sea necesario, integrando esta capacidad con el profiling de aplicacion (Clinic.js en Node, visto en el libro de NestJS) para un diagnostico completo de sistema a aplicacion.

## Errores comunes

- Escalar directamente a `perf`/`strace` sin haber usado primero herramientas de monitoreo mas simples que podrian haber identificado el problema mas rapido.
- Confundir un problema de syscalls lentas (diagnosticable con `strace`) con un problema de computo puro (diagnosticable con `perf`), usando la herramienta equivocada.
- No documentar sesiones de profiling profundo, perdiendo el conocimiento acumulado de como interpretar sus resultados para el equipo.
- No conectar el profiling a nivel de sistema operativo con herramientas especificas de la aplicacion (como Clinic.js para Node.js), perdiendo contexto completo.

## Practica

- [ ] Ejecuta `strace -c` sobre un comando simple e interpreta el resumen de syscalls.
- [ ] Adjunta `strace` a un proceso ya corriendo con `-p` y observa su actividad en tiempo real.
- [ ] Si `perf` esta disponible en tu sistema, usa `perf top` para ver funciones que consumen mas CPU ahora.
- [ ] Compara conceptualmente cuando usarias `strace` versus `perf` para un problema especifico.
- [ ] Anota como conectarias el profiling de sistema operativo con el profiling de aplicacion visto en el libro de NestJS.

## Preguntas de repaso

- Que tipo de problema es mas facil de diagnosticar con `strace` que con herramientas de monitoreo basico?
- Que diferencia hay entre lo que revela `strace` y lo que revela `perf`?
- Cuando escalarias de monitoreo basico a herramientas de profiling profundo como estas?
- Como se conecta el profiling de sistema operativo (`perf`) con el profiling especifico de una aplicacion Node.js (Clinic.js)?

## Cierre del libro

Completar este capitulo cierra el recorrido completo de Linux: desde entender que es el kernel y una distribucion, pasando por dominio de la terminal, permisos, procesos, redes, gestion de paquetes, almacenamiento, seguridad, administracion de servidores, hasta troubleshooting de nivel senior con herramientas de profiling profundo. El proximo paso natural es aplicar este conocimiento directamente en la administracion de los servidores donde corren tus proyectos reales, como [[NESTJS|el backend NestJS]] o los contenedores estudiados en [[DOCKER]].
