---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
Palabras clave:
  - IO, NIO, Files, Path, BufferedReader, FileWriter, InputStream, OutputStream
---

## 🧠 Concepto

Java tiene dos APIs para manejo de archivos:
- **java.io** — clásica, orientada a streams, bloqueante
- **java.nio** (NIO2, Java 7+) — moderna, `Path` y `Files`, mucho más simple

> [!tip] Preferir **NIO2** (`java.nio.file`) para código nuevo.

## NIO2 — java.nio.file (recomendado)

### Path — representar rutas

```java
Path ruta = Path.of("archivo.txt");
Path absoluta = Path.of("C:/Users/Juber/archivo.txt");
Path combinada = Path.of("C:/Users").resolve("Juber/archivo.txt");

ruta.getFileName();  // archivo.txt
ruta.getParent();    // directorio padre
ruta.toAbsolutePath(); // ruta absoluta
```

### Files — operaciones de archivos

```java
Path archivo = Path.of("datos.txt");

// Leer
String contenido = Files.readString(archivo); // Java 11+
List<String> lineas = Files.readAllLines(archivo);
byte[] bytes = Files.readAllBytes(archivo);

// Escribir
Files.writeString(archivo, "contenido"); // Java 11+
Files.write(archivo, "contenido".getBytes());
Files.write(archivo, lineas, StandardOpenOption.APPEND); // añadir

// Copiar / mover / borrar
Files.copy(origen, destino, StandardCopyOption.REPLACE_EXISTING);
Files.move(origen, destino, StandardCopyOption.REPLACE_EXISTING);
Files.delete(archivo);
Files.deleteIfExists(archivo);

// Verificar
Files.exists(archivo);
Files.isDirectory(archivo);
Files.isReadable(archivo);
Files.size(archivo); // en bytes

// Crear
Files.createFile(archivo);
Files.createDirectory(Path.of("nueva-carpeta"));
Files.createDirectories(Path.of("a/b/c")); // crea toda la jerarquía

// Listar directorio
Files.list(Path.of(".")).forEach(System.out::println);
Files.walk(Path.of("."), 2) // profundidad 2
    .filter(Files::isRegularFile)
    .forEach(System.out::println);
```

## BufferedReader / BufferedWriter (para archivos grandes)

```java
// Leer línea a línea
try (BufferedReader reader = Files.newBufferedReader(Path.of("archivo.txt"))) {
    String linea;
    while ((linea = reader.readLine()) != null) {
        System.out.println(linea);
    }
}

// O con streams (Java 8+)
try (Stream<String> lineas = Files.lines(Path.of("archivo.txt"))) {
    lineas.filter(l -> l.contains("error"))
          .forEach(System.out::println);
}

// Escribir
try (BufferedWriter writer = Files.newBufferedWriter(Path.of("salida.txt"))) {
    writer.write("Primera línea");
    writer.newLine();
    writer.write("Segunda línea");
}
```

## Serialización

```java
// Serializable — guardar objeto en archivo
public class Usuario implements Serializable {
    private static final long serialVersionUID = 1L;
    private String nombre;
    private int edad;
}

// Escribir
try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("usuario.ser"))) {
    oos.writeObject(new Usuario("Juber", 25));
}

// Leer
try (ObjectInputStream ois = new ObjectInputStream(
        new FileInputStream("usuario.ser"))) {
    Usuario usuario = (Usuario) ois.readObject();
}
```

## Try-with-resources

Siempre usar para cerrar streams automáticamente.

```java
// Antes (Java 6)
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("archivo.txt"));
    // ...
} finally {
    if (reader != null) reader.close();
}

// Con try-with-resources (Java 7+)
try (BufferedReader reader = new BufferedReader(new FileReader("archivo.txt"))) {
    // reader se cierra automáticamente al salir del bloque
}

// Múltiples recursos
try (InputStream in = Files.newInputStream(origen);
     OutputStream out = Files.newOutputStream(destino)) {
    in.transferTo(out);
}
```

## ⚠️ Errores Comunes

> [!warning] No cerrar streams
> Siempre usar `try-with-resources`. Un stream no cerrado puede bloquear el archivo.

> [!warning] `FileNotFoundException` vs `IOException`
> `FileNotFoundException` es subclase de `IOException`. Capturar la más específica primero.

## 📌 Notas Rápidas
