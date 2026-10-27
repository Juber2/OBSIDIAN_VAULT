---
tags:
  - Java
  - Patrones
  - Comportamiento
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - command, encapsular peticiones, undo, redo, cola de comandos, CQRS
---

## 🧠 Concepto

Encapsula una **petición como objeto**, permitiendo parametrizar clientes, hacer cola de peticiones, y soportar deshacer/rehacer.

**Cuándo usarlo:** Operaciones undo/redo, colas de tareas, transacciones, macros, CQRS.

## Implementación

```java
// Command (interfaz)
public interface Comando {
    void ejecutar();
    void deshacer();
}

// Receiver — el objeto que sabe hacer el trabajo
public class Editor {
    private StringBuilder texto = new StringBuilder();

    public void escribir(String s) { texto.append(s); }
    public void borrar(int cantidad) {
        int inicio = Math.max(0, texto.length() - cantidad);
        texto.delete(inicio, texto.length());
    }
    public String getTexto() { return texto.toString(); }
}

// Comandos concretos
public class ComandoEscribir implements Comando {
    private final Editor editor;
    private final String texto;

    public ComandoEscribir(Editor editor, String texto) {
        this.editor = editor;
        this.texto = texto;
    }

    public void ejecutar() { editor.escribir(texto); }
    public void deshacer() { editor.borrar(texto.length()); }
}

// Invoker — gestiona la ejecución y el historial
public class HistorialComandos {
    private final Deque<Comando> historial = new ArrayDeque<>();

    public void ejecutar(Comando comando) {
        comando.ejecutar();
        historial.push(comando);
    }

    public void deshacer() {
        if (!historial.isEmpty()) {
            historial.pop().deshacer();
        }
    }
}

// Uso
Editor editor = new Editor();
HistorialComandos historial = new HistorialComandos();

historial.ejecutar(new ComandoEscribir(editor, "Hola "));
historial.ejecutar(new ComandoEscribir(editor, "Mundo"));
System.out.println(editor.getTexto()); // "Hola Mundo"

historial.deshacer();
System.out.println(editor.getTexto()); // "Hola "
```

## Command como Functional Interface

```java
// Runnable / Callable ya son commands funcionales
Queue<Runnable> colaComandos = new LinkedList<>();
colaComandos.add(() -> System.out.println("Enviar email"));
colaComandos.add(() -> System.out.println("Actualizar inventario"));
colaComandos.add(() -> System.out.println("Notificar cliente"));

// Ejecutar todos
while (!colaComandos.isEmpty()) {
    colaComandos.poll().run();
}
```

## Relación con CQRS

En CQRS, los **Commands** son exactamente este patrón: objetos que encapsulan la intención de modificar el estado.

```java
// Command de CQRS
public record CrearUsuarioCommand(String nombre, String email) {}

// Handler
public class CrearUsuarioHandler {
    public void handle(CrearUsuarioCommand command) {
        // ejecutar la lógica
    }
}
```

## 📌 Notas Rápidas
- `Runnable`, `Callable` y cualquier functional interface que "encapsula una acción" son, en esencia, el patrón Command simplificado por lambdas.
- La parte de **deshacer** (undo) es lo que distingue a Command de simplemente pasar una función — si no necesitás undo/redo/cola de comandos, probablemente no necesitás el patrón completo, alcanza con un `Runnable`.
- En CQRS, todo Command (`CrearUsuarioCommand`, etc.) es este patrón aplicado a nivel de arquitectura: encapsula la intención de una operación como un objeto de datos inmutable.
- Útil para colas de trabajo diferido — encapsular la operación como objeto permite serializarla, loguearla o reintentarla más fácilmente que una llamada directa a método.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

