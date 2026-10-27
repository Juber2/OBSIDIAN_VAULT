---
tags:
  - Java
  - Patrones
  - Estructural
Creado: 2026-09-10
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - bridge, abstraccion, implementacion, explosion de clases, composicion, dimensiones
---

## 🧠 Concepto

Separa una **abstracción** de su **implementación** para que las dos puedan evolucionar por separado.

**Cuándo usarlo:** cuando tenés dos dimensiones que varían de forma independiente y la herencia te obliga a multiplicarlas.

## El problema: explosión de clases

Necesitás notificar al cliente. Hay tipos de notificación (movimiento, alerta de fraude, vencimiento) y canales (correo, SMS, push).

```
❌ Con herencia: una clase por combinación

NotificacionMovimientoEmail      NotificacionMovimientoSms      NotificacionMovimientoPush
NotificacionFraudeEmail          NotificacionFraudeSms          NotificacionFraudePush
NotificacionVencimientoEmail     NotificacionVencimientoSms     NotificacionVencimientoPush

3 tipos × 3 canales = 9 clases
Agregar WhatsApp → 12 clases
Agregar "aviso de cierre" → 16 clases
```

Crece **multiplicativamente**. Ese es el síntoma que Bridge resuelve.

## La solución: composición en vez de herencia

```
    Abstracción                    Implementación
   (qué se notifica)      →→→      (por dónde se envía)
                        el "puente"

    Notificacion                     CanalEnvio
        ├── Movimiento                   ├── Email
        ├── Fraude                       ├── Sms
        └── Vencimiento                  └── Push

    3 + 3 = 6 clases, y crece SUMANDO
```

## Implementación

```java
// La implementación: por dónde se envía
public interface CanalEnvio {
    void enviar(String destinatario, String asunto, String cuerpo);
}

public class CanalEmail implements CanalEnvio {
    public void enviar(String d, String asunto, String cuerpo) {
        mailSender.send(d, asunto, cuerpo);
    }
}

public class CanalSms implements CanalEnvio {
    public void enviar(String d, String asunto, String cuerpo) {
        smsGateway.send(d, truncar(cuerpo, 160));      // el SMS tiene su límite
    }
}

public class CanalPush implements CanalEnvio {
    public void enviar(String d, String asunto, String cuerpo) {
        pushService.send(d, asunto, cuerpo);
    }
}
```

```java
// La abstracción: qué se notifica
public abstract class Notificacion {

    protected final CanalEnvio canal;                  // ← el puente

    protected Notificacion(CanalEnvio canal) { this.canal = canal; }

    public abstract void notificar(Cliente cliente);
}

public class NotificacionMovimiento extends Notificacion {

    private final Movimiento movimiento;

    public NotificacionMovimiento(CanalEnvio canal, Movimiento m) {
        super(canal);
        this.movimiento = m;
    }

    @Override
    public void notificar(Cliente cliente) {
        canal.enviar(cliente.contacto(),
            "Movimiento en tu cuenta",
            "Se registró %s por %s".formatted(movimiento.tipo(), movimiento.monto()));
    }
}

public class NotificacionFraude extends Notificacion {
    @Override
    public void notificar(Cliente cliente) {
        canal.enviar(cliente.contacto(),
            "Operación bloqueada",
            "Detectamos una operación inusual. Si fuiste vos, confirmá desde la app.");
    }
}
```

```java
// Cualquier combinación, sin clases nuevas
new NotificacionMovimiento(new CanalSms(), movimiento).notificar(cliente);
new NotificacionFraude(new CanalPush(), ...).notificar(cliente);
```

> [!tip] El "puente" es literalmente el campo `canal`
> No hay nada más. El nombre del patrón viene de esa referencia que conecta las dos jerarquías y les permite variar por separado.

## Con Spring — la versión que vas a escribir

```java
public interface CanalEnvio {
    Canal tipo();
    void enviar(Mensaje mensaje);
}

@Service
public class ServicioNotificaciones {

    private final Map<Canal, CanalEnvio> canales;

    public ServicioNotificaciones(List<CanalEnvio> implementaciones) {
        this.canales = implementaciones.stream()
            .collect(toMap(CanalEnvio::tipo, identity()));      // Spring inyecta todas
    }

    public void notificar(Notificacion n, Cliente cliente) {
        Canal canal = cliente.canalPreferido();
        canales.get(canal).enviar(n.construirMensaje(cliente));
    }
}
```

Agregar WhatsApp es **una clase nueva con `@Component`**. Ni el servicio ni las notificaciones se tocan.

## 🆚 Bridge vs Strategy vs Adapter

Los tres usan composición y se confunden seguido.

| | Intención | Cuándo se decide |
|---|---|---|
| **[[11 - Strategy]]** | intercambiar **un algoritmo** | en ejecución, el cliente elige |
| **[[06 - Adapter]]** | hacer compatible **una interfaz ajena** | ya existía y no la podés cambiar |
| **Bridge** | separar **dos jerarquías** que varían | al diseñar, antes de que existan |

> [!tip] La diferencia con Strategy es la intención, no la estructura
> El código se parece mucho. Strategy varía **un** comportamiento dentro de una clase. Bridge separa **dos dimensiones completas** que van a crecer cada una por su lado.
> Si te preguntás cuál es, mirá si hay dos jerarquías que se multiplicarían: si sí, es Bridge.

> [!warning] Adapter se aplica después; Bridge, antes
> Adapter existe porque una interfaz ajena no encaja. Bridge se diseña de entrada, previendo que las dos dimensiones van a crecer.

## 🎯 Cuándo NO usarlo

| Situación | Por qué no |
|---|---|
| Solo una dimensión varía | alcanza con Strategy o polimorfismo simple |
| La implementación nunca va a cambiar | la indirección no compra nada |
| Dos combinaciones y no van a crecer | dos clases son más claras |

> [!danger] Es fácil aplicarlo de más
> Si tenés 2 tipos × 2 canales y no hay señal de que vayan a crecer, cuatro clases directas son más simples de leer que dos jerarquías y un puente. **Bridge se paga cuando la multiplicación duele**, no antes.

## 📌 Notas Rápidas
- El síntoma que lo delata: **la cantidad de clases crece multiplicando**. Si agregar una opción en una dimensión te obliga a crear N clases, es Bridge.
- En Java lo tenés delante todos los días: **JDBC** es Bridge puro — `Connection`, `Statement` y `ResultSet` son la abstracción, y cada driver (Postgres, MySQL, Oracle) es una implementación. Tu código no cambia al cambiar de base.
- También lo es SLF4J: la API de logging es la abstracción y Logback, Log4j2 o `java.util.logging` son implementaciones intercambiables.
- Con Spring casi no se escribe explícitamente: inyectar `List<Interfaz>` y elegir por clave logra lo mismo con menos ceremonia.
- Es de los patrones GoF menos usados a propósito, y de los más confundidos con [[11 - Strategy]].

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
