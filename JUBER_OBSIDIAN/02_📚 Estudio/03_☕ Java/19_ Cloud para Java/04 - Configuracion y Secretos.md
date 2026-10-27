---
tags:
  - Java
  - Cloud
  - Seguridad
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - 05 - Identidad y Acceso
Palabras clave:
  - Key Vault, Secrets Manager, Parameter Store, rotacion, 12-factor, feature flags
---

## 🧠 Concepto

Dos cosas distintas que se manejan distinto:

| | Configuración | Secreto |
|---|---|---|
| Ejemplo | URL de la base, timeouts, nivel de log | contraseñas, claves de API, certificados |
| ¿Puede ir al repositorio? | ✅ sí | ❌ **nunca** |
| Rotación | manual, ocasional | automática, periódica |
| Auditoría de acceso | no hace falta | ✅ obligatoria |
| Dónde vive | archivo, servicio de configuración | almacén de secretos cifrado |

> [!danger] Un secreto en el repositorio está comprometido para siempre
> Aunque lo borres en el commit siguiente, queda en la historia de Git, en los forks, en los clones locales de todo el equipo y en cualquier caché de la plataforma.
> La respuesta correcta a "se subió un secreto" nunca es "lo borré": es **rotar el secreto**.

## 📐 El principio

De la metodología *twelve-factor*: la configuración vive en el entorno, no en el código. Un mismo artefacto se despliega en desarrollo, pruebas y producción — **lo único que cambia es lo que se le inyecta**.

```java
// ❌ El entorno decide dentro del código
if (esProduccion()) { url = "prod-db"; } else { url = "localhost"; }

// ✅ El código no sabe dónde corre
@ConfigurationProperties(prefix = "corebank.db")
public record ConfigBaseDatos(String url, String usuario, String password) {}
```

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}      # se inyecta en ejecución, no está acá
```

## 🗄️ Los servicios

| Necesito | Azure | AWS |
|---|---|---|
| Secretos con rotación | **Key Vault** | **Secrets Manager** |
| Configuración simple y barata | App Configuration | **Parameter Store** |
| Feature flags | App Configuration | AppConfig |
| Certificados | Key Vault | ACM |
| Claves criptográficas | Key Vault (o Managed HSM) | KMS (o CloudHSM) |

> [!tip] AWS tiene dos servicios y la diferencia importa
> **Secrets Manager** cuesta por secreto y por mes, e incluye rotación automática con integración nativa a RDS. **Parameter Store** es prácticamente gratis en su nivel estándar, pero la rotación la implementás vos.
> Regla práctica: credenciales de base y claves sensibles en Secrets Manager; el resto de la configuración en Parameter Store.

## 💻 Integración con Spring

### Azure Key Vault

```xml
<dependency>
    <groupId>com.azure.spring</groupId>
    <artifactId>spring-cloud-azure-starter-keyvault-secrets</artifactId>
</dependency>
```

```yaml
spring:
  cloud:
    azure:
      keyvault:
        secret:
          property-sources:
            - endpoint: https://kv-corebank.vault.azure.net/
      credential:
        managed-identity-enabled: true       # ← sin credenciales en ninguna parte
```

Los secretos aparecen como propiedades normales: un secreto llamado `db-password` se lee con `${db-password}`.

### AWS Secrets Manager

```yaml
spring:
  config:
    import:
      - aws-secretsmanager:/corebank/produccion
      - aws-parameterstore:/corebank/produccion/
  cloud:
    aws:
      region:
        static: us-east-1
      # sin claves: usa el rol IAM de la tarea
```

> [!danger] Nunca pongas claves de acceso en la configuración
> ```yaml
> # ❌ Esto anula todo el modelo de seguridad
> aws:
>   credentials:
>     access-key: AKIA...
>     secret-key: wJal...
> ```
> La aplicación se autentica con **identidad gestionada** (Azure) o **rol IAM** (AWS): sin credenciales que guardar, rotar ni filtrar. Ver [[05 - Identidad y Acceso]].

## 🔄 Rotación

```
1 · Se genera la credencial nueva
2 · Coexisten ambas (ventana de solapamiento)
3 · Las aplicaciones toman la nueva
4 · Se revoca la vieja
```

> [!warning] Sin ventana de solapamiento, la rotación es una caída
> Si revocás la contraseña vieja en el mismo instante en que creás la nueva, toda instancia que todavía no releyó el secreto pierde la conexión. La rotación **tiene** que ser un proceso de dos fases.

### Releer sin reiniciar

```java
@Component
public class ProveedorDeSecretos {

    private final Cache<String, String> cache = Caffeine.newBuilder()
        .expireAfterWrite(Duration.ofMinutes(5))       // relee periódicamente
        .build();

    public String obtener(String nombre) {
        return cache.get(nombre, clienteSecretos::leer);
    }
}
```

| TTL | Consecuencia |
|---|---|
| Sin caché | una llamada por uso: lento y costoso |
| 5-15 min | ✅ equilibrio habitual |
| Solo al arrancar | la rotación exige redesplegar |

> [!tip] El pool de conexiones necesita atención especial
> HikariCP guarda la contraseña al crear cada conexión. Tras una rotación, las conexiones existentes siguen funcionando pero las nuevas fallan si el pool no releyó el secreto. La solución habitual es un `max-lifetime` menor al período de rotación, para que el pool se renueve solo.

```yaml
spring:
  datasource:
    hikari:
      max-lifetime: 1800000        # 30 min: las conexiones se reciclan
```

## 🚦 Feature flags

Cambiar comportamiento sin desplegar:

```java
@Service
public class ServicioTransferencia {

    private final FeatureManager features;

    public void transferir(ComandoTransferir cmd) {
        if (features.isEnabledAsync("nuevo-motor-antifraude").block()) {
            antifraudeV2.evaluar(cmd);
        } else {
            antifraudeV1.evaluar(cmd);
        }
    }
}
```

| Uso | Ejemplo |
|---|---|
| Despliegue progresivo | 5% de usuarios primero |
| Interruptor de emergencia | apagar una función que falla |
| Separar despliegue de activación | desplegar hoy, activar el lunes |
| Pruebas A/B | dos variantes en paralelo |

> [!warning] Los flags son deuda técnica con fecha de vencimiento
> Cada `if` sobre un flag duplica los caminos posibles del código y de los tests. Un sistema con veinte flags activos tiene un espacio de estados que nadie prueba entero.
> **Poné fecha de eliminación al crear el flag**, y borralo cuando la función esté estable.

## 📋 Jerarquía de configuración

```
1 · Valores por defecto en el código        (application.yml)
2 · Perfil                                   (application-prod.yml)
3 · Servicio de configuración                (App Configuration / Parameter Store)
4 · Almacén de secretos                      (Key Vault / Secrets Manager)
5 · Variables de entorno                     (mayor prioridad)
```

Spring resuelve en ese orden: lo de abajo pisa a lo de arriba.

> [!tip] Que la app falle al arrancar si falta una propiedad crítica
> ```java
> @ConfigurationProperties(prefix = "corebank.db")
> @Validated
> public record ConfigBaseDatos(
>         @NotBlank String url,
>         @NotBlank String usuario,
>         @NotBlank String password) {}
> ```
> Es mucho mejor que arranque roto y visible, a que arranque "bien" y falle en la primera petición con un `NullPointerException` que nadie relaciona con la configuración.

## 🌱 Desarrollo local

```yaml
# application-local.yml  ← este SÍ va al repositorio
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/corebank
    username: corebank
    password: local            # ✅ sin valor: es una base descartable
```

> [!danger] Nunca uses datos ni credenciales de producción en local
> Es la vía más directa a una filtración: una máquina de desarrollo no tiene los controles de un entorno productivo. Para probar con datos realistas, usá datos **anonimizados**.

Para probar la integración con el almacén de secretos:

```java
@Testcontainers
class SecretosIT {
    @Container
    static LocalStackContainer localstack = new LocalStackContainer(DockerImageName.parse("localstack/localstack"))
        .withServices(SECRETSMANAGER);
}
```

## 🔍 Detectar secretos filtrados

```yaml
- name: Detectar secretos
  uses: gitleaks/gitleaks-action@v2
```

```yaml
# .pre-commit-config.yaml — antes de que el commit exista
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks: [{ id: gitleaks }]
```

> [!tip] El gancho de pre-commit vale más que el escaneo en CI
> En CI el secreto **ya está** en la historia y hay que rotarlo. En el pre-commit ni siquiera llegó a existir. Los dos son útiles, pero el orden de valor es claro.

## 🧪 Tests

```java
@Test
void la_app_no_arranca_si_falta_una_propiedad_critica() {
    assertThrows(ApplicationContextException.class, () ->
        new SpringApplicationBuilder(App.class).properties("corebank.db.url=").run());
}

@Test
void ninguna_propiedad_sensible_se_expone_en_actuator() {
    var env = api.get("/actuator/env").as(Map.class);
    assertFalse(env.toString().contains(passwordReal));
}

@Test
void tras_rotar_el_secreto_las_conexiones_nuevas_usan_el_nuevo() { ... }

@Test
void ningun_secreto_aparece_en_los_logs_de_arranque() { ... }
```

> [!danger] `/actuator/env` expone la configuración
> Spring enmascara propiedades cuyo nombre contiene `password`, `secret`, `key` o `token`, pero **no** una llamada `corebank.db.credencial`. En producción, exponer `env` y `configprops` solo a redes internas — o directamente no exponerlos.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, prometheus       # env y configprops NO
```

## ⚠️ Errores Comunes

> [!warning] Los que producen filtraciones o caídas
> - **Secretos en el repositorio**: hay que rotar, no basta con borrar.
> - **Claves de acceso en la configuración** en vez de identidad gestionada.
> - **Secretos en variables de entorno de construcción**: quedan en la imagen.
> - **Rotación sin ventana de solapamiento**: caída garantizada.
> - **Sin caché de secretos**: latencia y costo por llamada.
> - **`max-lifetime` del pool mayor al período de rotación.**
> - **Exponer `/actuator/env`** públicamente.
> - **Datos de producción en desarrollo local.**
> - **Feature flags sin fecha de eliminación.**

## 📌 Notas Rápidas
- El indicador de que la configuración está bien resuelta: **el mismo artefacto se despliega en todos los entornos**. Si construís una imagen por entorno, la configuración está dentro del código.
- La combinación mínima recomendable: identidad gestionada para autenticarse, almacén de secretos para las credenciales, servicio de configuración para el resto, y validación al arrancar.
- Cuando se filtra un secreto, el orden es: rotar primero, investigar el alcance después, limpiar la historia al final. Invertir el orden deja la ventana abierta.
- Se aplica en [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[03 - Contenedores y Registries]]
- **Siguiente**: [[05 - Identidad y Acceso]]
- **Se conecta con**: [[05 - Configuracion y Properties]], [[01 - Key Vault y Managed Identity]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

