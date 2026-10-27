---
tags:
  - Java
  - Distribuido
  - Arquitectura
Creado: 2026-09-08
Relacionado:
  - Sistemas Distribuidos
  - 01 - Falacias de la Computacion Distribuida
Palabras clave:
  - CAP, PACELC, linealizabilidad, consistencia eventual, causal, particion, quorum
---

## 🧠 Concepto

El teorema **CAP** (Eric Brewer, 2000; demostrado por Gilbert y Lynch en 2002) enuncia que un sistema distribuido no puede garantizar simultáneamente:

| Letra | Significa | Definición precisa |
|---|---|---|
| **C** | Consistency | **linealizabilidad**: toda lectura ve la última escritura confirmada |
| **A** | Availability | todo nodo que funciona responde, sin error |
| **P** | Partition tolerance | el sistema sigue operando aunque se corte la comunicación entre nodos |

## ❌ El malentendido más común

> [!danger] CAP **no** es "elegí 2 de 3"
> Esa formulación es la que todo el mundo repite y está mal. Si tu sistema está distribuido, **la partición va a ocurrir**: los cables se cortan, los switches fallan, las zonas se aíslan. `P` no es una opción que se elige, es una condición del entorno.
>
> El enunciado correcto es: **durante una partición, tenés que elegir entre C y A.**

```
¿Hay partición?
├── NO  → podés tener C y A. CAP no dice nada acá.
└── SÍ  → elegí:
         ├── CP: rechazar peticiones que no podés garantizar consistentes
         └── AP: responder igual, aceptando devolver datos viejos
```

### El ejemplo concreto

```
Nodo A ←──✂️ partición ──→ Nodo B
saldo: 1000                saldo: 1000

Llega un retiro de 800 al nodo A.
A no puede consultar a B.

Opción CP → A rechaza: "no puedo garantizar consistencia" (pierde disponibilidad)
Opción AP → A acepta: registra el retiro (riesgo: B acepta otro y quedan en -600)
```

> [!tip] Para dinero, la respuesta casi siempre es CP
> Rechazar una operación es molesto; permitir un doble gasto es un agujero. Por eso las bases transaccionales de un core bancario son CP: ante partición, prefieren no responder antes que responder mal.
> Para un carrito de compras o un contador de "me gusta", AP es lo correcto: es preferible un dato levemente desactualizado a una página caída.

## 📐 PACELC — el complemento

CAP solo dice algo **durante** una partición, que es un evento raro. Daniel Abadi propuso **PACELC**, que cubre también el caso normal:

```
Si hay Partición  (P) → elegir entre Availability (A) y Consistency (C)
En caso contrario (E) → elegir entre Latency     (L) y Consistency (C)
```

La segunda mitad es la que importa el 99,9% del tiempo: **incluso sin partición, mantener la consistencia cuesta latencia**, porque hay que coordinar entre nodos antes de responder.

| Sistema | Clasificación | Lectura |
|---|---|---|
| PostgreSQL (una instancia) | — | no distribuido |
| PostgreSQL + réplica síncrona | PC/EC | consistente, más lento |
| PostgreSQL + réplica asíncrona | PC/EL | rápido, réplicas atrasadas |
| Cassandra | PA/EL | disponible y rápida, eventual |
| MongoDB (por defecto) | PC/EC | consistente en el primario |
| DynamoDB | PA/EL (configurable) | eventual por defecto |
| Spanner | PC/EC | consistente global, con costo de latencia |

> [!warning] La decisión suele ser configurable, no propiedad del producto
> Cassandra puede comportarse como CP subiendo el nivel de consistencia; Postgres con réplica asíncrona se comporta como EL. **La pregunta correcta no es "¿qué base uso?" sino "¿qué configuración uso para esta operación?"**.

## 📊 Modelos de consistencia

De más fuerte a más débil:

| Modelo | Garantía | Costo |
|---|---|---|
| **Linealizabilidad** | como si hubiera un solo nodo; orden real de tiempo | máximo |
| **Secuencial** | todos ven el mismo orden, no necesariamente el real | alto |
| **Causal** | lo relacionado por causa-efecto se ve en orden | medio |
| **Read-your-writes** | ves tus propias escrituras | bajo |
| **Lecturas monótonas** | no ves datos que retroceden | bajo |
| **Eventual** | si dejan de escribir, converge | mínimo |

### Consistencia eventual, en concreto

```
t=0   escribís  saldo = 1000  en el primario
t=1   leés del primario  → 1000  ✅
t=1   leés de la réplica → 950   ⚠️ todavía no replicó
t=3   leés de la réplica → 1000  ✅ convergió
```

El *lag* de replicación es real y medible. La pregunta de diseño es **cuánta desactualización tolera cada caso de uso**.

> [!danger] "Eventual" no significa "rápido"
> Bajo carga, un lag de réplica puede llegar a segundos o minutos. Un usuario que transfiere y de inmediato consulta su saldo puede ver el valor anterior — y va a reclamar. No es un bug de la base: es el modelo que elegiste.

### Read-your-writes: el mínimo aceptable para una UI

```java
@Service
public class ServicioConsulta {

    /** Tras escribir, este usuario lee del primario durante una ventana */
    public Dinero consultarSaldo(CuentaId cuenta, UUID usuario) {
        if (escriturasRecientes.tuvoEscritura(usuario, Duration.ofSeconds(5))) {
            return repoPrimario.saldo(cuenta);      // consistente
        }
        return repoReplica.saldo(cuenta);           // eventual, más barato
    }
}
```

> [!tip] El patrón "sticky" tras escritura resuelve el 90% de los reclamos
> El usuario que acaba de operar es el único que nota el lag, porque es el único que sabe qué escribió. Mandarlo al primario durante unos segundos elimina la percepción del problema sin renunciar a las réplicas para el resto del tráfico.

## 🗳️ Quórum

En sistemas replicados, la consistencia se ajusta con cuántas réplicas participan:

```
N = número de réplicas
W = réplicas que deben confirmar una escritura
R = réplicas que se consultan en una lectura

Si R + W > N  →  toda lectura ve la última escritura
```

| N | W | R | Comportamiento |
|---|---|---|---|
| 3 | 3 | 1 | escritura lenta, lectura rápida y consistente |
| 3 | 1 | 3 | escritura rápida, lectura lenta y consistente |
| 3 | 2 | 2 | **equilibrio típico**: tolera 1 nodo caído |
| 3 | 1 | 1 | máxima velocidad, consistencia eventual |

```java
// Cassandra: el nivel se elige por operación, no por base
session.execute(
    SimpleStatement.newInstance("SELECT saldo FROM cuentas WHERE id = ?", id)
        .setConsistencyLevel(ConsistencyLevel.QUORUM));    // R = mayoría
```

> [!tip] El nivel de consistencia se elige por operación
> Leer el saldo antes de una transferencia: `QUORUM`. Mostrar el histórico en un dashboard: `ONE`. Es la misma base y la misma tabla — cambia el requisito.

## 🎯 Cómo se decide en la práctica

Las preguntas, en orden:

1. **¿Qué pasa si esta lectura devuelve un dato de hace 2 segundos?** Si la respuesta es "nada grave", eventual alcanza.
2. **¿Qué pasa si se pierde una escritura confirmada?** Si es inaceptable, necesitás confirmación en quórum.
3. **¿Qué pasa si el sistema rechaza operaciones durante una partición?** Si es peor que dar un dato viejo, elegí AP.

| Caso | Elección | Por qué |
|---|---|---|
| Saldo antes de transferir | fuerte | doble gasto es inaceptable |
| Extracto histórico | eventual | los datos viejos no cambian |
| Catálogo de productos | eventual | un precio 2 s viejo no rompe nada |
| Reserva de stock | fuerte | no se puede vender dos veces |
| Contador de visitas | eventual | nadie audita el número |
| Registro de auditoría | fuerte + durable | es evidencia legal |

> [!warning] No apliques el mismo modelo a todo el sistema
> Es el error de diseño más caro: elegir "todo fuerte" cuesta latencia innecesaria en el 90% de las operaciones; elegir "todo eventual" mete inconsistencias donde no se toleran. **La granularidad correcta es el caso de uso, no la aplicación.**

## 🧪 Cómo se prueba

```java
@Test
void tras_escribir_el_mismo_usuario_lee_su_propio_valor() {
    servicio.depositar(cuenta, Dinero.de("500", PEN), usuario);
    assertEquals(Dinero.de("500", PEN), servicio.consultarSaldo(cuenta, usuario));
}

@Test
void con_la_replica_atrasada_la_lectura_critica_va_al_primario() {
    simularLagDeReplica(Duration.ofSeconds(10));
    assertEquals(saldoReal, servicio.saldoParaValidacion(cuenta));   // primario
}

@Test
void durante_una_particion_las_operaciones_de_dinero_se_rechazan() {
    toxiproxy.cortar("db-primaria");
    assertThrows(ServicioNoDisponibleException.class, () -> servicio.transferir(cmd));
    // Y NO: aceptar la operación con datos posiblemente viejos
}
```

Los tests de partición se hacen con **Toxiproxy** sobre Testcontainers, que permite cortar y restaurar la conectividad de forma reproducible.

## ⚠️ Errores Comunes

> [!warning] Los que aparecen en entrevistas y en producción
> - **Decir "elegí 2 de 3"**: es la formulación incorrecta de CAP.
> - **Creer que CAP dice algo cuando no hay partición**: no dice nada; para eso está PACELC.
> - **Aplicar un solo modelo a todo el sistema.**
> - **Leer de una réplica para validar una operación crítica**: el lag habilita el doble gasto.
> - **Suponer que "eventual" es "casi inmediato"**: bajo carga puede ser minutos.
> - **No medir el lag de replicación**: es una métrica que hay que tener siempre.
> - **Confundir consistencia de CAP con la C de ACID**: son cosas distintas — CAP habla de linealizabilidad entre réplicas, ACID de invariantes dentro de una transacción.

## 📌 Notas Rápidas
- La última confusión es sutil y frecuente: la **C de ACID** es "la transacción respeta las reglas de integridad"; la **C de CAP** es "todas las réplicas ven lo mismo". Aclararlo en una entrevista demuestra que entendiste los dos conceptos.
- El paper de Gilbert y Lynch (2002) es el que formalizó CAP. El artículo de Brewer de 2012, *"CAP Twelve Years Later"*, es el propio autor aclarando que "2 de 3" era una simplificación desafortunada.
- En la práctica, la mayoría de los sistemas bancarios son CP en el núcleo transaccional y AP en las capas de consulta y reportes. Es una decisión por capa, no por sistema.

## 🔗 Relaciones
- **Módulo**: [[Sistemas Distribuidos]]
- **Anterior**: [[01 - Falacias de la Computacion Distribuida]]
- **Siguiente**: [[03 - Idempotencia y Deduplicacion]]
- **Se conecta con**: [[13 - Consenso y Raft]], [[14 - Multiregion y DR]], [[05 - Base de Datos a Escala]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

