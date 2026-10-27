---
tags:
  - Java
  - Cloud
  - Datos
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - 01 - Modelo Mental Cloud
Palabras clave:
  - S3, Blob Storage, URL prefirmada, SAS, ciclo de vida, WORM, multipart, niveles
---

## 🧠 Concepto

El almacenamiento de objetos es donde van los archivos cuando el disco local no existe: comprobantes, documentos de identidad, reportes generados, respaldos, imágenes.

No es un sistema de archivos. Es un **almacén clave → valor** con metadatos, sin directorios reales y sin escrituras parciales.

| | Sistema de archivos | Almacenamiento de objetos |
|---|---|---|
| Estructura | árbol de directorios | plano; los "/" son parte del nombre |
| Modificar parte del archivo | ✅ | ❌ se reemplaza entero |
| Renombrar | instantáneo | copiar + borrar |
| Listar | rápido | costoso con muchos objetos |
| Escala | limitada | prácticamente ilimitada |
| Durabilidad | la del disco | 11 nueves |

> [!warning] "Carpetas" que no existen
> `documentos/2026/09/comprobante.pdf` es **un solo nombre de clave**, no tres niveles. Listar "la carpeta" es filtrar por prefijo, y con millones de objetos eso es lento y se factura.
> Diseñá los prefijos pensando en cómo vas a consultar, no en cómo se ve bonito.

## 🗄️ Los servicios

| | Azure | AWS |
|---|---|---|
| Servicio | **Blob Storage** | **S3** |
| Contenedor | container | bucket |
| Acceso temporal | SAS token | URL prefirmada |
| Inmutabilidad | immutable storage | Object Lock |
| Versionado | ✅ | ✅ |
| Eventos al escribir | Event Grid | notificaciones de S3 |

### Niveles de acceso

| Uso | Azure | AWS |
|---|---|---|
| Acceso frecuente | Hot | Standard |
| Poco frecuente (>30 días) | Cool | Standard-IA |
| Raro (>90 días) | Cold | Glacier Instant Retrieval |
| Archivo (recuperación en horas) | Archive | Glacier Flexible / Deep Archive |
| Automático | — | Intelligent-Tiering |

> [!danger] Los niveles fríos cobran por recuperar
> Guardar es barato; **leer es caro**, y en los niveles de archivo además hay que "rehidratar" el objeto y esperar horas. Un reporte que se consulta seguido no va en Archive por más que sea viejo.
> Además suelen tener permanencia mínima: borrar un objeto de Cool antes de 30 días se cobra igual que si hubiera estado los 30.

```java
// Mover a nivel frío después de N días
blobClient.setAccessTier(AccessTier.COOL);
```

## 🔑 El patrón fundamental: URLs temporales

**Nunca hagas pasar los archivos por tu aplicación.**

```java
// ❌ Todo el archivo atraviesa tu servicio: memoria, ancho de banda, timeouts
@PostMapping("/documentos")
public void subir(@RequestParam MultipartFile archivo) {
    blobClient.upload(archivo.getInputStream());     // 500 MB por tu JVM
}

// ✅ Tu servicio solo autoriza; el cliente sube directo al almacenamiento
@PostMapping("/documentos/url-de-subida")
public UrlSubida generarUrl(@RequestBody SolicitudSubida solicitud) {
    autorizar(solicitud);                             // ← tu lógica de negocio
    return almacen.urlDeSubida(claveDe(solicitud), Duration.ofMinutes(15));
}
```

```java
// Azure: SAS token
var permisos = new BlobSasPermission().setWritePermission(true);
var valores = new BlobServiceSasSignatureValues(
        OffsetDateTime.now().plusMinutes(15), permisos)
    .setContentType("application/pdf");              // restringe el tipo

String sas = blobClient.generateSas(valores);
String url = blobClient.getBlobUrl() + "?" + sas;
```

```java
// AWS: URL prefirmada
var solicitud = PutObjectRequest.builder()
    .bucket("corebank-documentos")
    .key(clave)
    .contentType("application/pdf")
    .build();

var prefirmada = presigner.presignPutObject(b -> b
    .signatureDuration(Duration.ofMinutes(15))
    .putObjectRequest(solicitud));

String url = prefirmada.url().toString();
```

> [!tip] Por qué importa tanto
> - Tu aplicación no consume memoria ni ancho de banda por archivos grandes.
> - No hay timeouts de HTTP por subidas lentas.
> - Escala sin límite: el almacenamiento aguanta lo que sea.
> - Menos costo de cómputo y de tráfico.
>
> **Es de las decisiones de arquitectura con mejor relación esfuerzo/beneficio en una aplicación cloud.**

> [!danger] La URL temporal es una credencial
> Cualquiera que la tenga puede subir o descargar hasta que expire. Reglas mínimas: vencimiento corto (minutos), permiso mínimo (solo escritura o solo lectura), clave fija por el servidor —nunca tomada del cliente— y restricción de tipo y tamaño.

```java
// ❌ El cliente elige la ruta: puede sobrescribir archivos ajenos
String clave = solicitud.rutaDelCliente();

// ✅ El servidor la construye
String clave = "clientes/%s/documentos/%s.pdf"
    .formatted(usuarioActual().clienteId(), UUID.randomUUID());
```

## 📦 Subidas grandes

```java
// El SDK divide en partes y las sube en paralelo, con reintento por parte
var transferManager = S3TransferManager.builder().s3Client(s3Async).build();

var subida = transferManager.uploadFile(b -> b
    .putObjectRequest(r -> r.bucket("corebank").key(clave))
    .source(Path.of("/tmp/reporte.csv")));

subida.completionFuture().join();
```

> [!warning] Las subidas incompletas se facturan
> Si una subida en partes se interrumpe, las partes ya cargadas quedan ocupando espacio **invisible**: no aparecen al listar objetos, pero se cobran. Una política de ciclo de vida que las limpie tras 7 días es obligatoria.

## ♻️ Ciclo de vida

```json
{
  "Rules": [
    {
      "Id": "documentos",
      "Filter": { "Prefix": "documentos/" },
      "Transitions": [
        { "Days": 30,  "StorageClass": "STANDARD_IA" },
        { "Days": 90,  "StorageClass": "GLACIER" }
      ],
      "Expiration": { "Days": 3650 }
    },
    {
      "Id": "limpiar-subidas-incompletas",
      "AbortIncompleteMultipartUpload": { "DaysAfterInitiation": 7 }
    }
  ]
}
```

> [!tip] Es la optimización de costo más rentable del almacenamiento
> Mover automáticamente lo viejo a niveles fríos y expirar lo que ya no hace falta reduce la factura sin tocar una línea de código. Configuralo al crear el bucket, no cuando la factura sorprenda.

## 🔒 Seguridad

### Nunca público

```bash
# AWS: bloquear acceso público a nivel de cuenta
aws s3control put-public-access-block --account-id 123456789 \
  --public-access-block-configuration \
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# Azure: prohibir acceso anónimo
az storage account update --name micuenta --allow-blob-public-access false
```

> [!danger] El bucket público es la filtración de datos más común de la nube
> Aparece en las noticias varias veces al año. El acceso a archivos se resuelve con **URLs temporales**, no abriendo el contenedor.

### Cifrado

| Nivel | Azure | AWS |
|---|---|---|
| Por defecto | claves de la plataforma | SSE-S3 |
| Clave del cliente | Key Vault + CMK | SSE-KMS |
| Cifrado del lado del cliente | SDK | SDK |

```java
// Cifrado con clave gestionada por el cliente
PutObjectRequest.builder()
    .bucket("corebank-documentos")
    .key(clave)
    .serverSideEncryption(ServerSideEncryption.AWS_KMS)
    .ssekmsKeyId("arn:aws:kms:us-east-1:123:key/abc-123")
    .build();
```

### Inmutabilidad (WORM)

```java
// AWS Object Lock: no se puede borrar ni modificar durante el período
PutObjectRequest.builder()
    .objectLockMode(ObjectLockMode.COMPLIANCE)          // ni el administrador puede
    .objectLockRetainUntilDate(Instant.now().plus(Duration.ofDays(3650)))
    .build();
```

> [!tip] WORM es el requisito para evidencia y auditoría
> En modo *compliance*, **nadie** puede borrar el objeto antes del vencimiento — ni la cuenta raíz. Es lo que convierte "prometemos no borrarlo" en un control técnico verificable, y lo que exige el regulador para registros de auditoría.
> Azure lo implementa con immutable storage (retención basada en tiempo o retención legal). Ver [[12 - Auditoria y Trazabilidad]].

## 🔔 Eventos

```
Se sube un archivo → evento → procesamiento automático
```

```java
// Se dispara cuando aparece un objeto con ese prefijo
@SqsListener("procesamiento-documentos")
public void alSubirArchivo(S3EventNotification evento) {
    for (var registro : evento.getRecords()) {
        String clave = registro.getS3().getObject().getKey();
        procesador.procesar(clave);
    }
}
```

Casos típicos: generar miniaturas, extraer texto de un PDF, validar un documento, disparar la conciliación cuando llega el archivo del corresponsal ([[09 - Conciliacion y Cuadre]]).

> [!warning] Las notificaciones son at-least-once
> El mismo evento puede llegar dos veces. Procesar un archivo dos veces puede duplicar registros — la idempotencia de [[03 - Idempotencia y Deduplicacion]] aplica igual acá.

## 🎯 Diseñar las claves

```
❌ documentos/comprobante-1.pdf          sin contexto ni particionamiento
❌ 2026-09-08/documentos/...             todo lo de un día en el mismo prefijo

✅ clientes/{clienteId}/documentos/{año}/{mes}/{uuid}.pdf
✅ conciliacion/{origen}/{fecha}/archivo.csv
```

| Criterio | Por qué |
|---|---|
| Empezar por la entidad | permite políticas de acceso por prefijo |
| Fecha en el medio | facilita el ciclo de vida y el archivado |
| UUID al final | evita colisiones |
| Sin datos personales en la clave | la clave aparece en logs y métricas |

> [!danger] La clave no es un lugar para datos sensibles
> `clientes/12345678/dni-juan-perez.pdf` filtra el documento de identidad y el nombre en cada log de acceso, en cada métrica y en cada URL. Los datos identificatorios van en los metadatos del objeto o en la base, no en la clave.

## 🧪 Tests

```java
@Test
void la_url_prefirmada_expira() {
    var url = almacen.urlDeSubida(clave, Duration.ofSeconds(2));
    esperar(Duration.ofSeconds(3));
    assertEquals(403, http.put(url, contenido).statusCode());
}

@Test
void la_url_de_subida_no_permite_leer_otros_objetos() {
    var url = almacen.urlDeSubida("clientes/A/doc.pdf", Duration.ofMinutes(5));
    assertEquals(403, http.get(url.replace("clientes/A", "clientes/B")).statusCode());
}

@Test
void el_cliente_no_puede_elegir_la_ruta_de_destino() {
    var respuesta = api.post("/documentos/url-de-subida",
        Map.of("ruta", "../../otro-cliente/doc.pdf"));
    assertFalse(respuesta.url().contains("otro-cliente"));
}

@Test
void el_bucket_no_permite_acceso_anonimo() {
    assertEquals(403, httpSinCredenciales.get(urlPublicaDelBucket).statusCode());
}

@Test
void un_objeto_con_object_lock_no_se_puede_borrar() { ... }
```

Con **LocalStack** o **Azurite** esto corre en Testcontainers sin tocar la nube.

## ⚠️ Errores Comunes

> [!warning] Los que cuestan dinero o datos
> - **Hacer pasar los archivos por la aplicación** en vez de usar URLs temporales.
> - **Bucket público**: la filtración clásica.
> - **El cliente elige la clave de destino**: sobrescribe archivos ajenos.
> - **URLs temporales de larga duración**: son credenciales.
> - **Sin límite de tipo y tamaño** en la subida.
> - **Datos personales en la clave**: se filtran en logs.
> - **Sin política de ciclo de vida**: la factura crece sin control.
> - **Sin limpieza de subidas incompletas**: se cobra espacio invisible.
> - **Guardar en niveles fríos** algo que se consulta seguido.
> - **Consumidor de eventos no idempotente**.

## 📌 Notas Rápidas
- Si te llevás una sola práctica: **URLs prefirmadas / SAS para subir y bajar**. Elimina de un saque los problemas de memoria, ancho de banda, timeouts y escala.
- El almacenamiento de objetos reemplaza al disco local, no a la base de datos. Los archivos van ahí; los metadatos —quién lo subió, cuándo, de qué operación— van en la base, con la clave del objeto como referencia.
- El modo *compliance* de Object Lock es lo que hace posible cumplir requisitos de retención de evidencia. Vale conocerlo por nombre para una entrevista de banca.
- Se aplica en [[Fase 3 - Seguridad y Auditoria]] y [[Fase 8 - Batch y Cierre de Dia]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[07 - Mensajeria Gestionada]]
- **Siguiente**: [[09 - Observabilidad Cloud]]
- **Se conecta con**: [[08 - IO y NIO]], [[05 - Azure SQL y Blob Storage]], [[12 - Auditoria y Trazabilidad]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

