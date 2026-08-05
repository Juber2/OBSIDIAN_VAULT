---
tags:
  - NestJS
  - TypeScript
  - Nest-GraphQL
Creado: 2026-07-27
Relacionado:
  - GraphQL y Frontend
  - NESTJS
---

# 05 - File Uploads y Streaming

## Que vas a aprender

En este capitulo vas a aprender a manejar **subida de archivos** en NestJS con `@nestjs/platform-express` y Multer, validacion de tipo/tamaño, y **streaming** de archivos grandes en vez de cargarlos completos en memoria.

## Explicacion

NestJS integra Multer para manejar `multipart/form-data` con interceptors (`FileInterceptor`, `FilesInterceptor`), que exponen el archivo subido como `Express.Multer.File` en el handler del controller. Es critico validar tipo MIME y tamaño **antes** de procesar el archivo, para evitar que un cliente suba contenido malicioso o archivos que agoten memoria o disco.

Para archivos grandes (videos, exports de datos), cargar todo el archivo en memoria antes de procesarlo es ineficiente y riesgoso; la alternativa es trabajar con **streams**, procesando el archivo en pedazos a medida que llega o se envia, sin necesitar tenerlo completo en memoria en ningun momento.

## Modelo mental

Pensa cargar un archivo completo en memoria como intentar tomar toda el agua de una manguera en un solo balde de una vez; un stream es dejar que el agua fluya de forma continua, procesando de a poco sin necesitar un balde del tamaño total.

## Ejemplo base

```typescript
@Post('upload')
@UseInterceptors(FileInterceptor('file', {
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
  fileFilter: (req, file, cb) => {
    if (!file.mimetype.match(/(jpg|jpeg|png)$/)) return cb(new BadRequestException('Formato invalido'), false);
    cb(null, true);
  },
}))
upload(@UploadedFile() file: Express.Multer.File) {
  return this.filesService.store(file);
}

@Get('download/:id')
async download(@Param('id') id: string, @Res() res: Response) {
  const stream = await this.filesService.getStream(id);
  stream.pipe(res); // streaming, no carga todo en memoria
}
```

## Como se usa en un proyecto real

1. Se limita el tamaño maximo de archivo y se valida el tipo MIME antes de procesar, nunca confiando en la extension del nombre de archivo.
2. Los archivos subidos se almacenan en almacenamiento externo (S3 o equivalente), no en el disco local del servidor, para no atar el estado de la aplicacion a una instancia especifica.
3. Se usan streams para archivos grandes tanto en subida como en descarga, evitando picos de uso de memoria.
4. Se escanean archivos subidos por usuarios contra malware cuando la aplicacion lo requiere por politica de seguridad.
5. Se generan URLs firmadas de corta duracion para descargas privadas en vez de exponer archivos publicamente sin control.

## Criterio junior

Un junior debe poder implementar un endpoint de subida de archivo simple con `FileInterceptor`, con limite de tamaño y validacion de tipo basica.

## Criterio mid

Un mid debe integrar la subida con almacenamiento externo (S3 o equivalente) y usar streaming para archivos de mayor tamaño en vez de cargarlos completos en memoria.

## Criterio senior

Un senior debe definir la politica de manejo de archivos de todo el sistema (limites, tipos permitidos, almacenamiento, escaneo de seguridad, URLs firmadas) y anticipar el impacto en memoria y disco de subidas concurrentes de muchos usuarios.

## Errores comunes

- Confiar solo en la extension del archivo para validar su tipo, en vez de verificar el MIME type real.
- Guardar archivos subidos en el disco local del servidor, rompiendo el sistema cuando se escala a multiples instancias.
- Cargar archivos grandes completos en memoria en vez de usar streams, arriesgando picos de memoria bajo carga.
- Exponer archivos privados con URLs publicas permanentes en vez de URLs firmadas con expiracion.

## Practica

- [ ] Implementa un endpoint de subida con limite de tamaño y validacion de tipo MIME.
- [ ] Integra la subida con un almacenamiento externo (S3 o simulado).
- [ ] Implementa descarga de un archivo usando streaming en vez de cargarlo completo.
- [ ] Investiga como generar una URL firmada de corta duracion para descarga privada.
- [ ] Anota que politica de archivos aplicarias en tu proyecto (tamaños, tipos, almacenamiento).

## Preguntas de repaso

- Por que no conviene confiar solo en la extension del archivo para validar su tipo?
- Que problema resuelve usar streaming en vez de cargar un archivo completo en memoria?
- Por que guardar archivos en el disco local del servidor es problematico al escalar horizontalmente?
- Que ventaja de seguridad da usar URLs firmadas de corta duracion para descargas privadas?
