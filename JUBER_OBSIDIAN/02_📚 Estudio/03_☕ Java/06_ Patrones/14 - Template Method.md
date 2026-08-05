---
tags:
  - Java
  - Patrones
  - Comportamiento
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - template method, algoritmo base, hook, pasos variables, clase abstracta
---

## 🧠 Concepto

Define el **esqueleto de un algoritmo** en una clase base, delegando algunos pasos a las subclases. Las subclases redefinen ciertos pasos sin cambiar la estructura general.

**Cuándo usarlo:** Algoritmos con pasos fijos y pasos variables, evitar duplicación de código en subclases.

## Implementación

```java
// Clase abstracta — define el template
public abstract class ProcesoImportacion {

    // Template Method — define el algoritmo
    public final void importar(String archivo) {
        abrirArchivo(archivo);
        validarFormato();
        procesarDatos();
        guardarEnBD();
        cerrarArchivo();
        if (debeNotificar()) { // hook
            notificarCompletado();
        }
    }

    // Paso fijo
    private void abrirArchivo(String archivo) {
        System.out.println("Abriendo: " + archivo);
    }

    // Pasos abstractos — las subclases DEBEN implementar
    protected abstract void validarFormato();
    protected abstract void procesarDatos();

    // Paso con implementación por defecto
    protected void guardarEnBD() {
        System.out.println("Guardando en BD...");
    }

    private void cerrarArchivo() {
        System.out.println("Cerrando archivo");
    }

    // Hook — las subclases PUEDEN sobreescribir (opcional)
    protected boolean debeNotificar() {
        return true; // por defecto sí notifica
    }

    protected void notificarCompletado() {
        System.out.println("Importación completada");
    }
}

// Implementación concreta para CSV
public class ImportacionCSV extends ProcesoImportacion {
    @Override
    protected void validarFormato() {
        System.out.println("Validando formato CSV...");
    }

    @Override
    protected void procesarDatos() {
        System.out.println("Procesando filas CSV...");
    }
}

// Implementación concreta para XML
public class ImportacionXML extends ProcesoImportacion {
    @Override
    protected void validarFormato() {
        System.out.println("Validando esquema XML...");
    }

    @Override
    protected void procesarDatos() {
        System.out.println("Parseando nodos XML...");
    }

    @Override
    protected boolean debeNotificar() {
        return false; // XML no notifica
    }
}

// Uso
ProcesoImportacion importacion = new ImportacionCSV();
importacion.importar("datos.csv");
```

## Template Method con lambdas (alternativa funcional)

```java
// Evitar herencia usando funciones
public class ProcesoImportacion {
    public void importar(
        String archivo,
        Runnable validar,
        Runnable procesar
    ) {
        System.out.println("Abriendo: " + archivo);
        validar.run();
        procesar.run();
        System.out.println("Guardando y cerrando");
    }
}

// Uso
new ProcesoImportacion().importar(
    "datos.csv",
    () -> System.out.println("Validando CSV"),
    () -> System.out.println("Procesando CSV")
);
```

## Template Method vs Strategy

| Template Method | Strategy |
|----------------|----------|
| Herencia | Composición |
| Modifica pasos del algoritmo | Cambia el algoritmo completo |
| Estructura fija | Estructura variable |
| Clase abstracta | Interfaz / lambda |

## 📌 Notas Rápidas
