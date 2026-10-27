---
tags:
  - Java
  - Cloud
  - FinOps
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - 12 - Infraestructura como Codigo
Palabras clave:
  - FinOps, egreso, reservas, spot, etiquetado, sobreaprovisionamiento, presupuesto
---

## 🧠 Concepto

En un datacenter propio, la infraestructura es una compra de capital que se decide una vez. En la nube es un **gasto operativo que tu código genera cada segundo**.

Eso convierte el costo en una responsabilidad de arquitectura: un bucle mal escrito, una consulta sin índice o un servicio en la región equivocada aparecen en la factura del mes.

> [!tip] FinOps no es "gastar menos"
> Es **gastar con intención**: saber qué se paga, por qué, y si el retorno lo justifica. Duplicar el gasto para cubrir el doble de clientes está perfecto; duplicarlo por un recurso olvidado, no.

## 💸 De dónde sale la factura

| Concepto | Peso típico | Comentario |
|---|---|---|
| Cómputo | 40-60% | instancias, contenedores, funciones |
| Bases de datos | 15-30% | suelen ser lo más caro por unidad |
| **Egreso de datos** | 5-20% | **el que más sorprende** |
| Almacenamiento | 5-15% | barato, pero se acumula |
| Observabilidad | 5-20% | puede dispararse sin control |
| Red (NAT, balanceadores) | 3-10% | invisible hasta que se mira |

### El egreso

```
Entrada de datos a la nube      →  gratis (casi siempre)
Salida hacia internet           →  se cobra por GB
Entre regiones                  →  se cobra
Entre zonas de disponibilidad   →  se cobra (menos)
Dentro de la misma zona         →  gratis
```

> [!danger] El egreso es el costo que nadie presupuesta
> Una API que devuelve respuestas de 2 MB y atiende 10 millones de peticiones al mes mueve 20 TB hacia internet. Es un costo que no aparece en ninguna estimación inicial y que crece exactamente con el éxito del producto.

| Mitigación | Ahorro |
|---|---|
| Comprimir respuestas (gzip, brotli) | 60-80% del volumen |
| Paginar y devolver solo los campos usados | alto |
| CDN para estáticos | alto |
| Endpoints privados en vez de NAT | alto |
| Servicios que se hablan mucho, en la misma zona | medio |

```yaml
server:
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html
    min-response-size: 1024
```

> [!tip] Activar compresión es una línea con retorno inmediato
> Un JSON comprime muy bien: reduce el egreso, la latencia percibida y el consumo de datos del cliente móvil, todo con una línea de configuración.

## 🏷️ Sin etiquetas no hay FinOps

```hcl
locals {
  etiquetas_comunes = {
    aplicacion   = "corebank"
    entorno      = var.entorno
    equipo       = "backend"
    centro_costo = "CC-1042"
    gestionado   = "terraform"
  }
}
```

> [!danger] Una factura sin etiquetas es un número que nadie puede accionar
> "Gastamos 40.000 dólares" no sirve para decidir nada. "El servicio de conciliación gastó 8.000, de los cuales 5.000 son egreso" sí.
> Las etiquetas se imponen desde el primer día por política — agregarlas a 300 recursos existentes es un proyecto.

```rego
deny[msg] {
  recurso := input.resource[_][_]
  not recurso.tags.centro_costo
  msg := "todo recurso requiere la etiqueta centro_costo"
}
```

## 📉 Los cinco desperdicios clásicos

### 1 · Recursos huérfanos

```bash
# Discos sin conectar
az disk list --query "[?diskState=='Unattached'].{nombre:name, gb:diskSizeGb}" -o table
aws ec2 describe-volumes --filters Name=status,Values=available

# IPs públicas reservadas y sin usar (se cobran igual)
az network public-ip list --query "[?ipConfiguration==null].name"
```

Instantáneas viejas, discos de máquinas borradas, balanceadores sin destinos, entornos de prueba que nadie apagó.

### 2 · Sobreaprovisionamiento

> [!warning] La instancia se dimensiona "por las dudas" y se queda así para siempre
> Se elige un tamaño generoso al lanzar, nunca se revisa, y termina al 8% de uso durante dos años.
> Las dos nubes tienen recomendaciones automáticas de redimensionamiento basadas en uso real. Revisarlas una vez por trimestre suele encontrar ahorros del 20-40%.

### 3 · Entornos que no duermen

```bash
# Apagar desarrollo fuera del horario laboral: ~65% de ahorro en esos recursos
0 20 * * 1-5   az vm deallocate --ids $VMS_DEV
0 7  * * 1-5   az vm start --ids $VMS_DEV
```

> [!tip] Desarrollo y pruebas no necesitan estar encendidos 24×7
> De 168 horas semanales, se usan unas 50. Apagar el resto ahorra cerca del 70% de esos entornos — y a nadie le molesta.

### 4 · Datos que nunca se archivan

```json
{
  "Rules": [{
    "Transitions": [
      { "Days": 30, "StorageClass": "STANDARD_IA" },
      { "Days": 90, "StorageClass": "GLACIER" }
    ],
    "Expiration": { "Days": 2555 }
  }]
}
```

Ver [[08 - Almacenamiento de Objetos]].

### 5 · Observabilidad sin control

> [!danger] Los logs pueden costar más que el cómputo que los genera
> `DEBUG` en producción, métricas con etiquetas de alta cardinalidad y trazas al 100% producen facturas que superan lo que cuesta correr la aplicación. Es sorprendentemente común.

| Control | Impacto |
|---|---|
| `INFO` en vez de `DEBUG` | muy alto |
| Muestreo de trazas al 1-10% | muy alto |
| Sin etiquetas de alta cardinalidad | muy alto |
| Retención corta para logs verbosos | alto |

Ver [[09 - Observabilidad Cloud]].

## 💳 Modelos de precio del cómputo

| Modelo | Descuento | Compromiso | Riesgo |
|---|---|---|---|
| Bajo demanda | — | ninguno | ninguno |
| **Reservas / planes de ahorro** | ~30-70% | 1-3 años | pagás aunque no uses |
| **Spot / puntual** | ~60-90% | ninguno | **te lo pueden quitar** |
| Nivel gratuito | — | — | límites bajos |

> [!tip] La combinación habitual
> **Reservas** para la carga base que sabés que va a estar (la que corre 24×7), **bajo demanda** para el pico variable, y **spot** para trabajos tolerantes a interrupción: procesamiento por lotes, CI, entornos de prueba.

> [!danger] Spot no sirve para servicios con estado
> La plataforma te avisa con unos segundos o pocos minutos y se lleva la instancia. Para una API de usuarios es inaceptable; para un job de conciliación que se puede reintentar, es un 80% de descuento gratis.
> Si usás spot, la aplicación **tiene** que manejar el aviso de terminación y apagar prolijo.

## 📊 Dónde se decide el costo en el código

| Decisión | Impacto |
|---|---|
| Consulta sin índice | 100× más CPU de base |
| N+1 | 100× más viajes y latencia |
| Sin caché en lecturas repetidas | carga y costo evitables |
| Respuestas sin comprimir ni paginar | egreso |
| Sondeo en bucle en vez de eventos | operaciones facturadas |
| Sin apagado ordenado | reintentos y trabajo duplicado |
| Métricas de alta cardinalidad | costo de observabilidad |

> [!tip] El mejor ahorro suele ser un índice
> Una consulta que hace `Seq Scan` sobre una tabla grande consume CPU de base, que es el recurso más caro por unidad. Arreglar la consulta puede permitir bajar el tamaño de la instancia, con ahorro directo y sostenido.
> **Optimizar código es optimizar factura** — ver [[03 - Indices y Queries Lentas]].

```java
@Cacheable(value = "tiposDeCambio", key = "#par")
public TipoDeCambio obtener(String par) {
    return clienteExterno.consultar(par);      // cada llamada se factura
}
```

## 🚨 Presupuestos y alertas

```bash
az consumption budget create \
  --budget-name presupuesto-corebank --amount 5000 --time-grain Monthly \
  --notifications '{"Alerta80": {"enabled": true, "operator": "GreaterThan",
                    "threshold": 80, "contactEmails": ["equipo@empresa.com"]}}'
```

| Alerta | Cuándo |
|---|---|
| 50%, 80%, 100% del presupuesto | mensual |
| Anomalía de gasto | pico inusual respecto de la tendencia |
| Recurso nuevo caro | al crearse |

> [!danger] La alerta de anomalía es la que salva
> Un bucle infinito que llama a una API externa, un job mal configurado que corre cada minuto en vez de cada día, un ataque que dispara el egreso. Con revisión mensual te enterás cuando ya son 30 días de gasto; con detección de anomalías, en horas.

## 🧮 Estimar antes de construir

```
API de transferencias, 1 M de peticiones/mes:

Cómputo    2 instancias × 2 vCPU × 4 GB × 730 h    ≈  X
Base       instancia mediana + 100 GB + réplica     ≈  Y
Egreso     1M × 5 KB = 5 GB                         ≈  Z (bajo)
Observ.    5 GB de logs + trazas al 10%             ≈  W
NAT        procesamiento de tráfico saliente        ≈  V
─────────────────────────────────────────────────────
Total mensual estimado
```

> [!tip] Estimá con las calculadoras oficiales, no de memoria
> Los precios cambian por región, por familia de instancia y por compromiso. Las calculadoras de Azure y AWS son gratuitas y dan un número defendible.
> Y estimá **antes** de elegir la arquitectura: descubrir después que la opción elegida cuesta el triple es una conversación incómoda.

## 🎯 Comparar arquitecturas por costo

| Escenario | Serverless | Contenedores |
|---|---|---|
| 10 mil peticiones/mes | centavos ✅ | costo de estar encendido |
| 1 millón/mes | comparable | comparable |
| 100 millones/mes | caro | mucho más barato ✅ |
| Tráfico esporádico | ✅ | desperdicio |
| Tráfico constante | desperdicio | ✅ |

> [!tip] Hay un punto de cruce, y conviene calcularlo
> Serverless cobra por uso; los contenedores, por estar encendidos. Con poco tráfico gana serverless; con tráfico constante gana el contenedor, por mucho. Hacé la cuenta con tu volumen real — ver [[11 - Serverless con Java]].

## ✅ Rutina de FinOps

**Mensual**
- [ ] Revisar la factura por etiqueta y por servicio.
- [ ] Buscar recursos huérfanos.
- [ ] Verificar que las alertas de presupuesto funcionaron.

**Trimestral**
- [ ] Revisar recomendaciones de redimensionamiento.
- [ ] Evaluar reservas para la carga base estable.
- [ ] Revisar retención de logs y políticas de ciclo de vida.
- [ ] Comparar costo por transacción contra el trimestre anterior.

**Continuo**
- [ ] Etiquetado obligatorio por política.
- [ ] Alerta de anomalías activa.
- [ ] Costo estimado en el pull request de infraestructura (`infracost`).

```yaml
- name: Estimar el impacto en costo del cambio
  uses: infracost/actions/comment@v3
  with:
    path: /tmp/infracost.json
```

> [!tip] Mostrar el costo en el pull request cambia el comportamiento
> Cuando quien propone la infraestructura ve "+ 340 USD/mes" en el comentario del PR, la conversación sobre el tamaño de la instancia ocurre **antes** de crearla, no seis meses después.

## ⚠️ Errores Comunes

> [!warning] Los que inflan la factura
> - **Sin etiquetas**: la factura no se puede atribuir a nada.
> - **Ignorar el egreso** al estimar.
> - **Entornos de desarrollo encendidos 24×7.**
> - **Recursos huérfanos** que nadie limpia.
> - **`DEBUG` en producción** y trazas al 100%.
> - **Métricas de alta cardinalidad.**
> - **Sobreaprovisionar "por las dudas"** y no revisar nunca.
> - **Reservas sin analizar el uso**: pagás capacidad que no usás.
> - **Spot para servicios con estado.**
> - **Todo el tráfico por NAT** pudiendo usar endpoints privados.
> - **Sin alerta de anomalías**: te enterás a fin de mes.

## 📌 Notas Rápidas
- La métrica que más ordena las decisiones es el **costo por transacción**. Si crece mientras el volumen crece, hay un problema de eficiencia; si baja, la arquitectura escala bien.
- El egreso y la observabilidad son los dos costos que más sorprenden, y los dos se controlan con decisiones simples: comprimir, paginar, muestrear, no usar `DEBUG`.
- Saber hablar de costos distingue a un backend senior de uno que solo entrega funcionalidad. En una entrevista, mencionar el punto de cruce entre serverless y contenedores, o el costo del egreso, demuestra que pensás el sistema completo.

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[12 - Infraestructura como Codigo]]
- **Siguiente**: [[14 - Equivalencias Azure AWS]]
- **Se conecta con**: [[Performance Java]], [[09 - Observabilidad Cloud]], [[11 - Serverless con Java]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

