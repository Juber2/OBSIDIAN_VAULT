---
tags:
  - Docker
  - DevOps
  - Docker-Volumenes
Creado: 2026-07-28
Relacionado:
  - Volumenes y Persistencia
  - DOCKER
---

# 03 - Backup y Restore de Volumenes

## Que vas a aprender

En este capitulo vas a aprender el patron estandar para hacer **backup y restore** del contenido de un volume Docker, usando un container temporal que monta el volume junto con un directorio del host.

## Explicacion

Docker no tiene un comando nativo "backup volume", pero el patron es simple y robusto: correr un container temporal y descartable (`--rm`) que monta el volume a respaldar en modo lectura, monta ademas un directorio del host donde guardar el backup, y ejecuta una herramienta estandar (`tar`) para comprimir el contenido del volume hacia ese directorio del host. El restore es el proceso inverso: un container temporal que descomprime un archivo del host dentro del volume de destino.

Este patron funciona igual sin importar que aplicacion use el volume (bases de datos, archivos de usuario), porque opera a nivel de filesystem, no de la aplicacion especifica. Para bases de datos, ademas conviene usar las herramientas nativas de backup del motor (`pg_dump`, `mysqldump`) cuando se necesita un backup logico consistente, en vez de solo copiar archivos crudos con el motor corriendo.

## Modelo mental

Pensa el backup de un volume como sacarle una foto completa a una caja de guardado y meter esa foto en un sobre etiquetado con la fecha, guardado en un lugar distinto de la caja original. El restore es literalmente reconstruir el contenido de una caja nueva a partir de esa foto guardada.

## Ejemplo base

```bash
# Backup: comprime el contenido del volume "datos-db" hacia ./backups del host
docker run --rm \
  -v datos-db:/origen:ro \
  -v $(pwd)/backups:/destino \
  alpine tar czf /destino/backup-$(date +%Y%m%d).tar.gz -C /origen .

# Restore: descomprime el backup dentro de un volume (nuevo o existente)
docker run --rm \
  -v datos-db-restaurado:/destino \
  -v $(pwd)/backups:/origen \
  alpine tar xzf /origen/backup-20260728.tar.gz -C /destino
```

## Como se usa en un proyecto real

1. Se automatiza el backup de volumenes criticos con un cron/tarea programada que ejecuta este patron periodicamente.
2. Se monta el volumen de origen en modo lectura (`:ro`) durante el backup, para evitar modificaciones accidentales durante el proceso.
3. Para bases de datos, se complementa (o reemplaza) el backup de filesystem con `pg_dump`/`mysqldump`, que da un backup logico consistente incluso con la base corriendo activamente.
4. Se prueban los restores periodicamente en un ambiente de prueba, no solo se confia en que el backup "deberia" funcionar sin haberlo verificado nunca.
5. Se envian los backups a almacenamiento externo (S3 o equivalente), no solo al mismo disco del servidor que aloja los volumenes originales.

## Criterio junior

Un junior debe poder ejecutar el patron basico de backup y restore de un volume siguiendo una guia, usando un container temporal con `tar`.

## Criterio mid

Un mid debe automatizar backups periodicos de volumenes criticos, y usar herramientas nativas de backup logico para bases de datos ademas del backup de filesystem.

## Criterio senior

Un senior debe diseñar la estrategia completa de backup y disaster recovery del sistema (frecuencia, retencion, almacenamiento externo, pruebas periodicas de restore) y definir el RPO/RTO aceptable para cada tipo de dato.

## Errores comunes

- Nunca probar un restore real, descubriendo recien durante un incidente real que el backup estaba corrupto o incompleto.
- Guardar backups en el mismo disco/servidor que los datos originales, perdiendo ambos ante una falla de ese servidor.
- Hacer backup de filesystem crudo de una base de datos corriendo sin usar las herramientas nativas de backup logico, arriesgando un backup inconsistente.
- No automatizar el proceso, dependiendo de que alguien se acuerde de correrlo manualmente.

## Practica

- [ ] Ejecuta el patron de backup para un volume de practica con datos de ejemplo.
- [ ] Restaura ese backup en un volume nuevo y verifica que los datos son identicos.
- [ ] Automatiza el backup con una tarea programada simple.
- [ ] Investiga como complementar el backup con `pg_dump` si tu volumen contiene una base Postgres.
- [ ] Define el RPO (cuanto dato podes permitirte perder) y RTO (cuanto tiempo de recuperacion es aceptable) para tu sistema.

## Preguntas de repaso

- Por que Docker no tiene un comando nativo de backup de volumenes, y como se resuelve en la practica?
- Por que es importante probar el proceso de restore, no solo el de backup?
- Que ventaja da un backup logico (`pg_dump`) sobre copiar archivos crudos de una base de datos corriendo?
- Por que los backups no deberian guardarse en el mismo servidor que los datos originales?
