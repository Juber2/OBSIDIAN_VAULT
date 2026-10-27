---
tags:
  - Java
  - Cloud
  - Docker
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - 01 - Docker con Java
Palabras clave:
  - Dockerfile, multi-stage, ACR, ECR, jib, capas, CVE, firma, digest
---

## 🧠 Concepto

Una imagen de contenedor es el **artefacto desplegable**: contiene tu aplicación, sus dependencias y su runtime. El *registry* es donde vive.

Lo que hace bien una imagen: que sea chica (se descarga rápido), reproducible (la misma imagen en todos lados), segura (sin vulnerabilidades conocidas, sin root) y trazable (se sabe de qué commit salió).

## 🏗️ El Dockerfile

```dockerfile
# ── Etapa de construcción ────────────────────────────────
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app

# Las dependencias primero: esta capa se cachea entre builds
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline -B

# El código después: cambia siempre, invalida solo desde acá
COPY src ./src
RUN ./mvnw clean package -DskipTests -B

# ── Etapa de ejecución ───────────────────────────────────
FROM eclipse-temurin:21-jre-alpine

RUN addgroup -S app && adduser -S app -G app

WORKDIR /app
COPY --from=build --chown=app:app /app/target/*.jar app.jar

USER app                                   # ← nunca root
EXPOSE 8080

ENV JAVA_OPTS="-XX:MaxRAMPercentage=75 -XX:+UseG1GC -XX:+ExitOnOutOfMemoryError"
ENTRYPOINT ["sh", "-c", "exec java $JAVA_OPTS -jar app.jar"]
```

> [!tip] El orden de las capas define la velocidad de tus builds
> Docker cachea por capa e invalida todas las posteriores al primer cambio. Copiar el `pom.xml` y descargar dependencias **antes** de copiar el código hace que un cambio en una clase no vuelva a descargar medio Maven Central.
> Con el orden invertido, cada build descarga todo de nuevo: minutos en vez de segundos.

> [!danger] `exec` en el ENTRYPOINT no es opcional
> Sin `exec`, el shell es el PID 1 y **no propaga `SIGTERM` a la JVM**. El resultado: la aplicación nunca ejecuta su apagado ordenado y el orquestador la mata a la fuerza tras el timeout. Peticiones cortadas en cada despliegue.

### Tamaño

| Imagen base | Tamaño aproximado | Comentario |
|---|---|---|
| `eclipse-temurin:21-jdk` | ~450 MB | JDK completo: no va en producción |
| `eclipse-temurin:21-jre` | ~270 MB | razonable |
| `eclipse-temurin:21-jre-alpine` | ~180 MB | ✅ buen equilibrio |
| JRE recortado con `jlink` | ~90 MB | requiere trabajo extra |
| Imagen nativa + `distroless` | ~80 MB | ver [[11 - Serverless con Java]] |

```dockerfile
# jlink: un runtime con solo los módulos que usás
RUN jlink --add-modules $(jdeps --print-module-deps app.jar) \
          --strip-debug --no-man-pages --no-header-files --compress=2 \
          --output /jre
```

> [!warning] Alpine usa musl en vez de glibc
> Es lo que la hace chica, y también lo que puede romper bibliotecas con código nativo. Si algo falla de forma extraña, probá una base `-jammy` o `-noble` antes de perder horas: el ahorro de 80 MB rara vez vale el problema.

### Jib — sin Dockerfile

```xml
<plugin>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>jib-maven-plugin</artifactId>
    <configuration>
        <from><image>eclipse-temurin:21-jre-alpine</image></from>
        <to><image>acr.azurecr.io/corebank:${project.version}</image></to>
        <container>
            <user>1000:1000</user>
            <jvmFlags>
                <jvmFlag>-XX:MaxRAMPercentage=75</jvmFlag>
                <jvmFlag>-XX:+ExitOnOutOfMemoryError</jvmFlag>
            </jvmFlags>
        </container>
    </configuration>
</plugin>
```

```bash
./mvnw compile jib:build      # construye y publica, sin demonio de Docker
```

> [!tip] Jib separa dependencias, recursos y clases en capas distintas
> Como las dependencias cambian rara vez y tu código cambia siempre, un despliegue sube solo unos kilobytes en vez de la imagen entera. Además no necesita Docker instalado, lo que simplifica bastante los pipelines de CI.

## 🏷️ Etiquetado

```bash
# ❌ No dice qué está desplegado ni permite volver atrás
docker push acr.azurecr.io/corebank:latest

# ✅ Trazable
docker push acr.azurecr.io/corebank:1.4.2
docker push acr.azurecr.io/corebank:1.4.2-a3f9c1e     # versión + commit
```

> [!danger] `latest` en producción es un problema de trazabilidad
> No sabés qué versión está corriendo, no podés volver atrás con certeza, y dos instancias arrancadas en momentos distintos pueden tener **imágenes diferentes con la misma etiqueta**.

### El digest: la referencia inmutable

```yaml
# Etiqueta: puede reasignarse a otra imagen
image: acr.azurecr.io/corebank:1.4.2

# Digest: identifica el contenido exacto, no se puede reasignar
image: acr.azurecr.io/corebank@sha256:a3f9c1e4b8d2...
```

> [!tip] Las etiquetas son mutables, el digest no
> Cualquiera puede volver a publicar `1.4.2` apuntando a otro contenido. El digest es un hash del contenido: si cambia el contenido, cambia el digest. Para despliegues reproducibles y auditables, referenciá por digest.

## 📦 Los registries

| | Azure | AWS |
|---|---|---|
| Servicio | Container Registry (ACR) | Elastic Container Registry (ECR) |
| Autenticación | Entra ID / Managed Identity | IAM |
| Escaneo de vulnerabilidades | Microsoft Defender | Amazon Inspector |
| Replicación geográfica | ✅ (nivel Premium) | replicación entre regiones |
| Firma de imágenes | Notation / Notary | ECR + Cosign |

```bash
# Azure: sin contraseñas, con la identidad del usuario o del servicio
az acr login --name miacr
docker push miacr.azurecr.io/corebank:1.4.2

# AWS
aws ecr get-login-password --region us-east-1 \
  | docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com
```

> [!danger] Nunca guardes credenciales del registry en el pipeline
> Usá identidad federada: GitHub Actions con OIDC obtiene un token temporal sin que exista ningún secreto de larga vida guardado en el repositorio. Es más seguro y una cosa menos que rotar — ver [[05 - Identidad y Acceso]].

```yaml
# GitHub Actions con OIDC: sin secretos almacenados
permissions:
  id-token: write
  contents: read

steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ vars.AZURE_CLIENT_ID }}
      tenant-id: ${{ vars.AZURE_TENANT_ID }}
      subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
```

### Políticas de retención

```bash
# ACR: borrar imágenes sin etiqueta de más de 7 días
az acr config retention update --registry miacr --status enabled --days 7 --type UntaggedManifests
```

Un registry sin política de limpieza crece indefinidamente y se factura por almacenamiento. Es de los costos que nadie mira hasta que ya es grande.

## 🔒 Seguridad

### Escaneo de vulnerabilidades

```yaml
- name: Escanear la imagen
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: corebank:${{ github.sha }}
    severity: CRITICAL,HIGH
    exit-code: 1              # falla el build
```

> [!warning] Escaneá en el pipeline **y** en el registry
> En el pipeline detectás lo que traés hoy. En el registry detectás las vulnerabilidades **descubiertas después** de publicar: una imagen que era limpia ayer puede tener un CVE crítico mañana sin que nadie la haya tocado.

### Checklist de imagen

- [ ] Multi-stage: el JDK y el código fuente no están en la imagen final.
- [ ] Usuario no root (`USER app`).
- [ ] Sin secretos en capas ni en variables de entorno de construcción.
- [ ] Base actualizada y con versión fijada.
- [ ] Sin herramientas innecesarias (`curl`, `bash`, compiladores).
- [ ] Escaneo sin CVE críticos.
- [ ] Sistema de archivos de solo lectura donde se pueda.
- [ ] Etiquetada con versión y commit; referenciada por digest.

> [!danger] Un secreto en una capa queda ahí para siempre
> ```dockerfile
> COPY .env /app/.env
> RUN rm /app/.env          # ❌ el archivo sigue en la capa anterior
> ```
> Borrar en una capa posterior **no elimina** el contenido de la anterior: cualquiera que descargue la imagen puede extraerlo. Los secretos se inyectan en ejecución, nunca en construcción — ver [[04 - Configuracion y Secretos]].

```dockerfile
# Si necesitás un secreto DURANTE el build (token de un repo privado)
RUN --mount=type=secret,id=maven_token \
    ./mvnw -s settings.xml package        # no queda en ninguna capa
```

### Procedencia y firma

```bash
cosign sign --key azurekms://... acr.azurecr.io/corebank@sha256:a3f9...
cosign verify --key azurekms://... acr.azurecr.io/corebank@sha256:a3f9...
```

Firmar permite que el clúster rechace imágenes que no vengan de tu pipeline. Junto con un SBOM (inventario de componentes) es la base de la seguridad de la cadena de suministro.

## 🧪 Verificaciones

```bash
# ¿Corre como no root?
docker run --rm corebank:1.4.2 id
# uid=1000(app) gid=1000(app)   ✅

# ¿Respeta el límite de memoria?
docker run --rm -m 512m corebank:1.4.2 \
  java -XX:+PrintFlagsFinal -version | grep MaxHeapSize
# debe dar ~384 MB (75% de 512), no el 25% de la RAM del host

# ¿Propaga SIGTERM?
docker run -d --name t corebank:1.4.2 && docker stop t
docker logs t | grep -i "graceful shutdown"      # debe aparecer

# ¿Cuánto pesa?
docker images corebank:1.4.2 --format "{{.Size}}"
```

```java
@Test
void la_imagen_arranca_y_responde_el_health() {
    try (var contenedor = new GenericContainer<>("corebank:" + version)
            .withExposedPorts(8080)
            .waitingFor(Wait.forHttp("/actuator/health").forStatusCode(200))) {
        contenedor.start();
        assertTrue(contenedor.isRunning());
    }
}
```

## ⚠️ Errores Comunes

> [!warning] Los que se pagan en cada despliegue
> - **Sin multi-stage**: el JDK y el código fuente viajan a producción.
> - **Correr como root.**
> - **`latest` en producción**: sin trazabilidad ni vuelta atrás confiable.
> - **Secretos en capas**: quedan para siempre.
> - **`ENTRYPOINT` sin `exec`**: no se propaga `SIGTERM`.
> - **Copiar el código antes de las dependencias**: builds lentísimos.
> - **Sin escaneo de vulnerabilidades.**
> - **Credenciales del registry como secretos** en vez de OIDC.
> - **Sin política de retención**: el registry crece sin límite.

## 📌 Notas Rápidas
- **Jib** es la opción más simple para Java: sin Dockerfile, sin demonio de Docker, con capas bien separadas por defecto. Si el proyecto no tiene necesidades raras, es lo primero que probaría.
- El truco del orden de capas (dependencias antes que código) es la optimización de mayor impacto en el tiempo de build, y la que más seguido falta.
- Referenciar por digest en vez de por etiqueta convierte el despliegue en reproducible de verdad. Es un cambio de una línea con bastante retorno.
- Se aplica en [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[02 - Computo para Apps Java]]
- **Siguiente**: [[04 - Configuracion y Secretos]]
- **Se conecta con**: [[01 - Docker con Java]], [[03 - CI CD para Java]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

