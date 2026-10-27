---
tags:
  - Java
  - Cloud
  - DevOps
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - 03 - CI CD para Java
Palabras clave:
  - Terraform, Bicep, CloudFormation, CDK, estado, deriva, plan, modulos, politicas
---

## 🧠 Concepto

**Infraestructura como código**: la nube se define en archivos versionados, se revisa en un pull request y se aplica con una herramienta. No se hace clic en un portal.

| Con clics | Con código |
|---|---|
| Nadie sabe qué cambió ni quién | historial en Git |
| Los entornos divergen | idénticos por construcción |
| Recrear lleva días | un comando |
| Sin revisión | pull request |
| Se documenta aparte (y queda viejo) | el código **es** la documentación |

> [!danger] El cambio manual en el portal es el enemigo del modelo
> Alguien "ajusta rapidito" una regla de firewall a las 2 AM durante un incidente. Ese cambio no está en el código. La próxima vez que se aplique la infraestructura, **se revierte** — y el incidente vuelve.
> Si hay que tocar a mano por urgencia, el primer trabajo del día siguiente es llevarlo al código.

## 🛠️ Las herramientas

| Herramienta | Alcance | Lenguaje |
|---|---|---|
| **Terraform** | multi-nube | HCL declarativo |
| **OpenTofu** | multi-nube | HCL (bifurcación de Terraform) |
| **Bicep** | Azure | DSL declarativo |
| ARM | Azure | JSON |
| **CloudFormation** | AWS | YAML/JSON |
| **CDK** | AWS (y CDKTF) | TypeScript, Java, Python |
| **Pulumi** | multi-nube | lenguajes generales |

> [!tip] Si vas a aprender una sola, que sea Terraform
> Funciona en las dos nubes que cubre este módulo, es lo más pedido en las ofertas de trabajo, y el modelo mental (estado, plan, aplicar) se transfiere a todas las demás.
> Bicep es más agradable si trabajás solo en Azure; CDK es cómodo si preferís un lenguaje de programación real.

## 📄 Un ejemplo

```hcl
# main.tf
terraform {
  required_version = "~> 1.9"
  required_providers {
    azurerm = { source = "hashicorp/azurerm", version = "~> 4.0" }
  }
  backend "azurerm" {                       # ← el estado, remoto y bloqueado
    resource_group_name  = "rg-tfstate"
    storage_account_name = "sttfstatecorebank"
    container_name       = "tfstate"
    key                  = "corebank.tfstate"
  }
}

resource "azurerm_postgresql_flexible_server" "db" {
  name                = "psql-corebank-${var.entorno}"
  resource_group_name = azurerm_resource_group.rg.name
  location            = var.ubicacion
  version             = "16"
  sku_name            = var.sku_base
  storage_mb          = var.almacenamiento_mb

  backup_retention_days        = var.entorno == "prod" ? 35 : 7
  geo_redundant_backup_enabled = var.entorno == "prod"
  zone                         = "1"
  high_availability {
    mode                      = var.entorno == "prod" ? "ZoneRedundant" : "SameZone"
    standby_availability_zone = "2"
  }

  tags = local.etiquetas_comunes
}
```

```hcl
# El mismo módulo para todos los entornos: solo cambian las variables
module "corebank" {
  source   = "./modulos/corebank"
  entorno  = "prod"
  sku_base = "GP_Standard_D4s_v3"
}
```

> [!tip] Un módulo, muchos entornos
> Es lo que garantiza que producción y preproducción sean iguales. Si cada entorno tiene su propio archivo copiado y pegado, **van a divergir** — y el bug que solo aparece en producción vendrá de ahí.

## 💾 El estado

Terraform guarda un archivo de estado que mapea el código a los recursos reales. Es la pieza más delicada.

```hcl
backend "s3" {
  bucket         = "corebank-tfstate"
  key            = "prod/terraform.tfstate"
  region         = "us-east-1"
  encrypt        = true
  dynamodb_table = "terraform-locks"      # ← bloqueo entre ejecuciones
}
```

> [!danger] Tres reglas del estado, todas obligatorias
> 1. **Remoto**, nunca en la máquina de alguien ni en Git.
> 2. **Bloqueado**, para que dos personas no apliquen a la vez y lo corrompan.
> 3. **Con versionado y respaldo**: si se pierde, Terraform no sabe qué recursos administra y el próximo `apply` intenta **crear todo de nuevo**.

> [!warning] El estado contiene secretos
> Contraseñas generadas, cadenas de conexión, claves: todo queda en texto plano dentro del archivo de estado. Por eso va cifrado, con acceso restringido, y **nunca** en un repositorio.

### La deriva

```bash
terraform plan     # compara: código vs estado vs realidad
```

```
# azurerm_postgresql_flexible_server.db se modificó fuera de Terraform
~ backup_retention_days = 7 -> 35
```

> [!tip] Ejecutá `plan` periódicamente aunque no cambies nada
> Detecta los cambios manuales antes de que sorprendan en el próximo despliegue. Un `plan` programado que alerta cuando hay diferencias es una de las mejoras baratas más útiles.

## 🔄 El flujo

```
1 · Rama y cambio en el código
2 · Pull request
3 · CI ejecuta `plan` y lo publica como comentario
4 · Revisión humana del plan          ← acá está el valor
5 · Aprobación y merge
6 · CD ejecuta `apply`
```

```yaml
name: infra
on: [pull_request]

jobs:
  plan:
    runs-on: ubuntu-latest
    permissions:
      id-token: write            # OIDC: sin secretos almacenados
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}

      - run: terraform init
      - run: terraform validate
      - run: terraform plan -out=tfplan -no-color | tee plan.txt

      - name: Publicar el plan en el PR
        uses: actions/github-script@v7
        with:
          script: |
            const plan = require('fs').readFileSync('plan.txt', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner, repo: context.repo.repo,
              body: '```\n' + plan.slice(0, 60000) + '\n```'
            });
```

> [!danger] Leé el plan antes de aprobar
> Es el único momento en que ves qué se va a destruir. Un cambio de nombre en un recurso puede traducirse a `destroy` + `create` sobre una base de datos de producción.
> ```
> # azurerm_postgresql_flexible_server.db debe reemplazarse
> -/+ resource "azurerm_postgresql_flexible_server" "db" {
>       ~ name = "psql-corebank" -> "psql-corebank-prod"  # obliga a reemplazar
> ```
> **Ese `-/+` es la destrucción de una base de datos.** Aprobar sin leer es cómo se pierden datos.

```hcl
# Red de seguridad para lo crítico
resource "azurerm_postgresql_flexible_server" "db" {
  lifecycle {
    prevent_destroy = true
  }
}
```

## 🔒 Seguridad y políticas

```yaml
- name: Analizar la infraestructura
  uses: bridgecrewio/checkov-action@master
  with:
    directory: infra/
    soft_fail: false
```

Detectan cosas como: buckets públicos, bases sin cifrado, security groups abiertos a `0.0.0.0/0`, logs desactivados, secretos en el código.

```rego
# Política como código: bloquear bases sin alta disponibilidad en producción
deny[msg] {
  input.resource.azurerm_postgresql_flexible_server[name]
  input.variables.entorno == "prod"
  not input.resource.azurerm_postgresql_flexible_server[name].high_availability
  msg := sprintf("la base %v en prod requiere alta disponibilidad", [name])
}
```

> [!tip] La política como código convierte una convención en un control
> "Acordamos que en producción todo lleva alta disponibilidad" es una convención que alguien va a olvidar. Una política que falla el pipeline es un control que no se puede saltear por descuido.

## 🏷️ Etiquetado

```hcl
locals {
  etiquetas_comunes = {
    aplicacion   = "corebank"
    entorno      = var.entorno
    equipo       = "backend"
    centro_costo = var.centro_costo
    gestionado   = "terraform"
    repositorio  = "miorg/corebank-infra"
  }
}
```

> [!warning] Sin etiquetas no hay control de costos posible
> Con doscientos recursos sin etiquetar, la factura es un número global que nadie puede atribuir. Las etiquetas son lo que permite responder "¿cuánto cuesta este servicio?" — ver [[13 - Costos y FinOps]].
> La etiqueta `gestionado = terraform` además distingue lo que administra el código de lo creado a mano.

## 🧪 Probar la infraestructura

```go
// Terratest: despliega de verdad, verifica y destruye
func TestBaseDeDatos(t *testing.T) {
    opciones := &terraform.Options{
        TerraformDir: "../modulos/corebank",
        Vars: map[string]interface{}{"entorno": "test"},
    }
    defer terraform.Destroy(t, opciones)
    terraform.InitAndApply(t, opciones)

    endpoint := terraform.Output(t, opciones, "db_endpoint")
    assert.NotEmpty(t, endpoint)
    assert.False(t, esAccesibleDesdeInternet(endpoint))
}
```

```bash
terraform fmt -check -recursive     # formato
terraform validate                  # sintaxis
tflint                              # buenas prácticas
checkov -d infra/                   # seguridad
```

## 📦 Organización

```
infra/
├── modulos/
│   ├── red/
│   ├── base-datos/
│   ├── aplicacion/
│   └── observabilidad/
├── entornos/
│   ├── dev/       terraform.tfvars
│   ├── qa/        terraform.tfvars
│   └── prod/      terraform.tfvars
└── politicas/
```

| Práctica | Por qué |
|---|---|
| Estado separado por entorno | un error en dev no toca prod |
| Módulos reutilizables | los entornos no divergen |
| Versiones de proveedores fijadas | builds reproducibles |
| Sin secretos en `.tfvars` | vienen del almacén de secretos |

> [!danger] Estados separados por entorno, sin excepción
> Con un estado compartido, un `apply` mal apuntado en desarrollo puede destruir recursos de producción. La separación de estados es la barrera que evita ese error, y es barata.

## ⚠️ Errores Comunes

> [!warning] Los que destruyen infraestructura
> - **Aprobar un `plan` sin leerlo**: el `-/+` borra bases de datos.
> - **Estado local o en Git**: se pierde o se corrompe, y contiene secretos.
> - **Sin bloqueo de estado**: dos `apply` simultáneos lo corrompen.
> - **Estado compartido entre entornos.**
> - **Cambios manuales en el portal**: deriva silenciosa.
> - **Copiar y pegar entre entornos** en vez de usar módulos.
> - **Secretos en `.tfvars`** o en variables del código.
> - **Sin `prevent_destroy`** en recursos con datos.
> - **Sin etiquetas**: costos inatribuibles.
> - **Versiones de proveedores sin fijar**: el mismo código da resultados distintos.

## 📌 Notas Rápidas
- La regla que hace funcionar todo esto: **la nube se toca solo desde el código**. En cuanto se acepta el clic manual "por esta vez", el modelo empieza a erosionarse.
- Leer el `plan` es la habilidad más importante y la que menos se practica. El `-/+` (reemplazar) es la señal a la que hay que prestar atención: significa destruir y volver a crear.
- El estado con secretos en texto plano sorprende a mucha gente. Es una de las razones por las que el backend remoto cifrado no es opcional.
- Se aplica en [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[11 - Serverless con Java]]
- **Siguiente**: [[13 - Costos y FinOps]]
- **Se conecta con**: [[03 - CI CD para Java]], [[06 - SRE APM y IaC]], [[05 - Configuracion de Ambientes]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

