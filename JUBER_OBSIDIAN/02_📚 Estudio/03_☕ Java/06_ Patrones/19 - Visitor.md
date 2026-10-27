---
tags:
  - Java
  - Patrones
  - Comportamiento
Creado: 2026-09-10
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - visitor, doble despacho, jerarquia, operaciones externas, pattern matching, sealed
---

## 🧠 Concepto

Permite agregar **operaciones nuevas** sobre una jerarquía de clases **sin modificar esas clases**.

**Cuándo usarlo:** la jerarquía es estable pero las operaciones cambian seguido. En Java moderno, el pattern matching sobre tipos sellados lo reemplaza en casi todos los casos.

## El problema

```java
public sealed interface Evento permits Deposito, Retiro, Transferencia { }
```

Sobre esos eventos necesitás: generar apuntes contables, calcular comisiones, exportar a CSV, validar límites, generar el texto del extracto…

```java
// ❌ Meter cada operación dentro de la jerarquía la infla
public record Deposito(...) implements Evento {
    List<Apunte> generarApuntes() { ... }
    Dinero calcularComision() { ... }
    String aCsv() { ... }
    String textoExtracto() { ... }         // el dominio se llena de responsabilidades ajenas
}
```

El dominio termina sabiendo de contabilidad, de CSV y de presentación. Cada operación nueva toca **todas** las clases.

## Implementación clásica

```java
public interface VisitanteEvento<R> {
    R visitar(Deposito d);
    R visitar(Retiro r);
    R visitar(Transferencia t);
}

public interface Evento {
    <R> R aceptar(VisitanteEvento<R> visitante);        // ← el "accept"
}

public record Deposito(CuentaId cuenta, Dinero monto) implements Evento {
    @Override public <R> R aceptar(VisitanteEvento<R> v) { return v.visitar(this); }
}

public record Retiro(CuentaId cuenta, Dinero monto, Canal canal) implements Evento {
    @Override public <R> R aceptar(VisitanteEvento<R> v) { return v.visitar(this); }
}
```

```java
// Una operación = un visitante. La jerarquía no se toca.
public class GeneradorDeApuntes implements VisitanteEvento<List<Apunte>> {

    @Override public List<Apunte> visitar(Deposito d) {
        return List.of(new Apunte(CAJA, d.monto().negado()),
                       new Apunte(d.cuenta(), d.monto()));
    }

    @Override public List<Apunte> visitar(Retiro r) {
        var apuntes = new ArrayList<Apunte>();
        apuntes.add(new Apunte(r.cuenta(), r.monto().negado()));
        apuntes.add(new Apunte(CAJA, r.monto()));
        if (r.canal() == Canal.CAJERO_AJENO) {
            apuntes.add(new Apunte(COMISIONES, COMISION_CAJERO));
        }
        return apuntes;
    }

    @Override public List<Apunte> visitar(Transferencia t) { ... }
}
```

```java
List<Apunte> apuntes = evento.aceptar(new GeneradorDeApuntes());
Dinero comision     = evento.aceptar(new CalculadoraDeComisiones());
String linea        = evento.aceptar(new ExportadorCsv());
```

## 🔀 El doble despacho

Es el mecanismo que hace funcionar el patrón, y es lo que se pregunta en entrevistas.

```
1 · evento.aceptar(visitante)     → despacha según el tipo REAL del evento
2 · visitante.visitar(this)       → despacha según el tipo del VISITANTE
```

Java resuelve la sobrecarga de métodos en **tiempo de compilación**, mirando el tipo declarado:

```java
// ❌ Esto NO funciona como esperás
public void procesar(Evento e) { ... }
public void procesar(Deposito d) { ... }

Evento e = new Deposito(...);
procesar(e);                    // llama a procesar(Evento), NO a procesar(Deposito)
```

El `aceptar()` resuelve eso: dentro de `Deposito.aceptar()`, `this` **es** un `Deposito` en tiempo de compilación, así que `v.visitar(this)` elige la sobrecarga correcta.

## ✅ La versión moderna: pattern matching

```java
public List<Apunte> generarApuntes(Evento evento) {
    return switch (evento) {
        case Deposito(var cuenta, var monto) ->
            List.of(new Apunte(CAJA, monto.negado()), new Apunte(cuenta, monto));

        case Retiro(var cuenta, var monto, Canal.CAJERO_AJENO) ->
            List.of(new Apunte(cuenta, monto.negado()),
                    new Apunte(CAJA, monto),
                    new Apunte(COMISIONES, COMISION_CAJERO));

        case Retiro(var cuenta, var monto, var canal) ->
            List.of(new Apunte(cuenta, monto.negado()), new Apunte(CAJA, monto));

        case Transferencia(var origen, var destino, var monto) ->
            List.of(new Apunte(origen, monto.negado()), new Apunte(destino, monto));
    };
}
```

| | Visitor clásico | `sealed` + pattern matching |
|---|---|---|
| Código necesario | interfaz + `aceptar` en cada clase | solo el `switch` |
| Agregar una operación | una clase visitante | un método |
| Agregar un tipo | tocar la interfaz y **todos** los visitantes | el compilador señala cada `switch` |
| Verificación de exhaustividad | ❌ manual | ✅ del compilador |
| Legibilidad | dispersa en varias clases | todo junto |

> [!tip] En Java 21+, empezá por el `switch`
> Con `sealed` el compilador garantiza exhaustividad, que es la propiedad principal que Visitor buscaba, y sin la ceremonia del `aceptar()`. Ver [[13 - Pattern Matching Moderno]].
> **Nunca pongas `default`** en ese `switch`: desactiva la verificación y el día que agregues un tipo nadie te avisa.

## 🎯 Cuándo el Visitor clásico sigue ganando

| Situación | Por qué |
|---|---|
| No podés sellar la jerarquía | sin `sealed` no hay exhaustividad |
| La jerarquía es de una biblioteca ajena | no podés agregarle `permits` |
| Java anterior a 21 | no hay patrones en `switch` |
| El visitante mantiene estado entre visitas | un acumulador es más natural en una clase |
| Recorrido de árboles (AST, XML) | el visitante controla el recorrido |

```java
// Visitante con estado: acumula mientras recorre
public class TotalizadorPorMoneda implements VisitanteEvento<Void> {
    private final Map<Moneda, Dinero> totales = new EnumMap<>(Moneda.class);

    @Override public Void visitar(Deposito d) {
        totales.merge(d.monto().moneda(), d.monto(), Dinero::mas);
        return null;
    }
    public Map<Moneda, Dinero> resultado() { return Map.copyOf(totales); }
}
```

## 🌳 El uso donde sigue siendo insustituible

Recorrer un árbol de sintaxis: compiladores, parsers, motores de reglas.

```java
public interface VisitanteExpresion<R> {
    R visitar(Literal l);
    R visitar(Suma s);
    R visitar(Comparacion c);
    R visitar(Y y);
}

// Una gramática, muchas operaciones: evaluar, imprimir, optimizar, traducir a SQL
public class TraductorASql implements VisitanteExpresion<String> {
    @Override public String visitar(Comparacion c) {
        return "%s %s ?".formatted(c.campo(), c.operador().sql());
    }
    @Override public String visitar(Y y) {
        return "(%s AND %s)".formatted(y.izq().aceptar(this), y.der().aceptar(this));
    }
}
```

Es el motor de un [[21 - Specification]] traducido a SQL, o de un motor de reglas de antifraude.

## 📌 Notas Rápidas
- La regla de decisión: **¿qué cambia más seguido, los tipos o las operaciones?** Si cambian las operaciones y los tipos son estables, Visitor (o su versión moderna). Si cambian los tipos, polimorfismo normal.
- El **doble despacho** es el concepto que hay que poder explicar: Java resuelve la sobrecarga por tipo declarado en compilación, y el `aceptar()` es el truco para conseguir despacho por tipo real.
- En Java moderno lo reemplaza `sealed` + pattern matching en la mayoría de los casos, y con **mejor** garantía: el compilador verifica exhaustividad, cosa que el Visitor clásico nunca hizo.
- Donde sigue vivo: recorrido de árboles y visitantes con estado acumulado. Ahí la clase es más natural que un `switch`.
- Es probablemente el patrón GoF más difícil de leer y el que menos se usa en un backend de negocio.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
