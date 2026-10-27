---
tags:
  - Java
  - Cloud
  - Seguridad
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - 04 - Configuracion y Secretos
Palabras clave:
  - Managed Identity, IAM Role, OIDC, workload identity, minimo privilegio, RBAC, IRSA
---

## 🧠 Concepto

El problema: tu aplicación necesita leer un secreto, escribir en un bucket y publicar en una cola. **¿Cómo prueba quién es?**

La respuesta ingenua —una clave de API guardada en algún lado— crea el problema que quería resolver: ahora hay una credencial de larga vida que alguien tiene que guardar, rotar y proteger.

La respuesta correcta: **la plataforma le da identidad al recurso**, y la credencial nunca existe.

```
❌ App → lee la clave de un archivo → se autentica → usa el servicio
✅ App → la plataforma le inyecta un token temporal → usa el servicio
```

## 🆔 Los mecanismos

| Concepto | Azure | AWS |
|---|---|---|
| Identidad del recurso | **Managed Identity** | **IAM Role** |
| Asignada por el sistema | System-assigned | rol de instancia / de tarea |
| Reutilizable | User-assigned | rol asumible |
| Para Kubernetes | Workload Identity | IRSA |
| Para CI/CD externo | Federación OIDC | OIDC provider |
| Modelo de permisos | RBAC de Azure | políticas IAM |

### Azure — Managed Identity

```bash
az containerapp identity assign --name corebank --resource-group rg --system-assigned

# Se le da permiso sobre el recurso concreto
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/.../vaults/kv-corebank
```

```java
// La aplicación no maneja ninguna credencial
var credencial = new DefaultAzureCredentialBuilder().build();

var cliente = new SecretClientBuilder()
    .vaultUrl("https://kv-corebank.vault.azure.net/")
    .credential(credencial)
    .buildClient();

String password = cliente.getSecret("db-password").getValue();
```

> [!tip] `DefaultAzureCredential` prueba varios métodos en orden
> En producción usa la identidad gestionada; en tu máquina usa la sesión de `az login` o las variables del IDE. **El mismo código funciona en los dos lados** sin condicionales ni perfiles.

### AWS — IAM Roles

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["secretsmanager:GetSecretValue"],
    "Resource": "arn:aws:secretsmanager:us-east-1:123456789:secret:corebank/prod-*"
  }]
}
```

```java
// El SDK obtiene credenciales temporales del rol de la tarea
var cliente = SecretsManagerClient.builder()
    .credentialsProvider(DefaultCredentialsProvider.create())
    .build();
```

| Contexto | De dónde salen las credenciales |
|---|---|
| EC2 | perfil de instancia (IMDSv2) |
| ECS | rol de la tarea |
| Lambda | rol de ejecución |
| EKS | IRSA (rol vía cuenta de servicio) |

## ☸️ En Kubernetes

El pod se autentica con un token de su cuenta de servicio, que la nube canjea por credenciales del rol.

```yaml
# AWS — IRSA
apiVersion: v1
kind: ServiceAccount
metadata:
  name: corebank
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/corebank-role
```

```yaml
# Azure — Workload Identity
apiVersion: v1
kind: ServiceAccount
metadata:
  name: corebank
  annotations:
    azure.workload.identity/client-id: "a3f9c1e4-..."
---
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    metadata:
      labels:
        azure.workload.identity/use: "true"
    spec:
      serviceAccountName: corebank
```

> [!danger] Un secreto de Kubernetes no está cifrado por defecto
> Los `Secret` se guardan en etcd **codificados en base64**, que no es cifrado. Cualquiera con acceso de lectura a los secretos del namespace los ve en claro.
> Las salidas: habilitar cifrado en reposo de etcd, o —mejor— no guardarlos ahí y leerlos del almacén de secretos con identidad de carga de trabajo.

## 🔗 CI/CD sin secretos

El caso más olvidado: el pipeline también necesita identidad para desplegar.

```yaml
# GitHub Actions con OIDC: cero secretos almacenados
permissions:
  id-token: write
  contents: read

jobs:
  desplegar:
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
```

```yaml
      # AWS
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/github-deploy
          aws-region: us-east-1
```

> [!tip] La confianza se restringe al repositorio y a la rama
> ```json
> "StringLike": {
>   "token.actions.githubusercontent.com:sub": "repo:miorg/corebank:ref:refs/heads/main"
> }
> ```
> Sin esa condición, **cualquier repositorio de GitHub** podría asumir tu rol. Es un error de configuración con consecuencias graves y bastante frecuente.

## 🎯 Mínimo privilegio

```json
// ❌ Permite leer todos los secretos de la cuenta
{ "Effect": "Allow", "Action": "secretsmanager:*", "Resource": "*" }

// ✅ Solo lo que necesita, sobre lo que necesita
{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "arn:aws:secretsmanager:us-east-1:123:secret:corebank/prod/db-*"
}
```

| Antipatrón | Por qué es un problema |
|---|---|
| `Action: "*"` | permite borrar, no solo leer |
| `Resource: "*"` | alcanza a recursos de otros equipos |
| Roles compartidos entre servicios | un servicio comprometido accede a todo |
| Permisos "temporales" que quedan | nadie los revisa después |

> [!tip] Empezá sin permisos y agregá lo que falle
> Es más rápido y mucho más seguro que empezar amplio y recortar: el recorte nunca se hace, porque nadie quiere romper algo que funciona.
> Las dos nubes tienen herramientas que analizan el uso real y sugieren la política mínima. Vale correrlas periódicamente.

### Separá identidades por servicio

```
❌ Una identidad "app" con permisos para todo
✅ corebank-api        → leer secretos de la base, publicar en la cola
   corebank-batch      → leer secretos, leer y escribir en el bucket
   corebank-consumidor → leer de la cola, escribir en la base
```

Si comprometen uno, el alcance queda acotado a lo que ese componente podía hacer.

## 🧑 Identidad de la aplicación vs del usuario

Dos planos que no hay que mezclar:

| | Identidad de la carga de trabajo | Identidad del usuario |
|---|---|---|
| Quién es | el servicio | la persona |
| Autentica ante | la nube | tu aplicación |
| Mecanismo | Managed Identity / IAM Role | JWT, OAuth2, sesión |
| Autoriza | acceso a recursos de infraestructura | acceso a datos de negocio |
| Cubierto en | esta nota | [[Spring Security]] |

> [!danger] Que la app pueda leer un dato no significa que el usuario pueda
> La identidad de la carga de trabajo le da a tu servicio permiso para consultar la base. La autorización de negocio —que **este** usuario pueda ver **esta** cuenta— es responsabilidad de tu código.
> Confundirlas produce el fallo IDOR de [[Fase 3 - Seguridad y Auditoria]]: el servicio tiene permiso, y devuelve datos de cualquiera.

## 🔐 Cifrado en reposo

```
Nube → clave gestionada por la plataforma      (por defecto, sin trabajo)
     → clave gestionada por el cliente (CMK)   (control y auditoría)
     → clave propia del cliente (BYOK/HYOK)    (control total, más operación)
```

| Nivel | Azure | AWS |
|---|---|---|
| Por defecto | claves de la plataforma | claves de AWS |
| Cliente | Key Vault + CMK | KMS con clave propia |
| Hardware dedicado | Managed HSM | CloudHSM |

> [!tip] Para banca suele exigirse clave gestionada por el cliente
> No por criptografía —el algoritmo es el mismo— sino por **control y auditoría**: querés poder revocar la clave, ver quién la usó y demostrar que el proveedor no puede descifrar sin tu autorización. Ver [[11 - PCI-DSS y HSM]].

## 🧪 Tests y verificaciones

```java
@Test
void la_app_se_autentica_sin_credenciales_en_configuracion() {
    var props = contexto.getEnvironment();
    assertNull(props.getProperty("aws.credentials.access-key"));
    assertNull(props.getProperty("azure.client-secret"));
}

@Test
void la_app_no_puede_leer_secretos_de_otro_servicio() {
    assertThrows(AccessDeniedException.class,
        () -> clienteSecretos.leer("otro-servicio/prod/db-password"));
}
```

```bash
# ¿Hay claves de larga vida en la cuenta?
aws iam list-users --query 'Users[?PasswordLastUsed==null]'
aws iam list-access-keys --user-name ci-deploy      # debería estar vacío

# ¿Qué permisos tiene realmente la identidad?
az role assignment list --assignee $PRINCIPAL_ID --output table
```

## ⚠️ Errores Comunes

> [!warning] Los que abren la puerta
> - **Claves de larga vida** en vez de identidad gestionada.
> - **Confianza OIDC sin restringir repositorio y rama**: cualquiera asume tu rol.
> - **`Action: "*"` o `Resource: "*"`.**
> - **Una identidad compartida** por todos los servicios.
> - **Secretos de Kubernetes** creyendo que base64 es cifrado.
> - **Confundir identidad de carga de trabajo con autorización de negocio** → IDOR.
> - **Permisos temporales que nunca se revocan.**
> - **Credenciales del pipeline como secretos de repositorio** en vez de OIDC.

## 📌 Notas Rápidas
- La frase que resume el módulo: **identidad en vez de credenciales**. Si en algún lugar hay una clave guardada, hay una pregunta pendiente sobre por qué.
- `DefaultAzureCredential` y `DefaultCredentialsProvider` son la pieza que hace esto cómodo: el mismo código funciona en local y en producción sin ramificaciones.
- La federación OIDC para CI/CD es la mejora de seguridad con mejor relación costo-beneficio que se puede hacer en un pipeline: elimina el secreto de despliegue, que suele ser el más poderoso del repositorio.
- Se aplica en [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[04 - Configuracion y Secretos]]
- **Siguiente**: [[06 - Bases de Datos Gestionadas]]
- **Se conecta con**: [[01 - Key Vault y Managed Identity]], [[04 - Entra ID y Seguridad de APIs]], [[Spring Security]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

