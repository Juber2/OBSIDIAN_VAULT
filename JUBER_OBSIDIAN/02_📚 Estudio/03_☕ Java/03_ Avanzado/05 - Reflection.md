---
tags:
  - Java
  - Avanzado
Creado: 2026-05-21
Relacionado:
  - Java Avanzado
  - Annotations
Palabras clave:
  - reflection, Class, Method, Field, Constructor, introspección, dynamic proxy
---

## 🧠 Concepto

**Reflection** permite inspeccionar y manipular clases, métodos, campos y constructores en tiempo de ejecución. Es la base de frameworks como Spring, Hibernate y JUnit.

> [!note] Usar con moderación — Reflection rompe encapsulamiento y es más lento que el acceso directo.

## Obtener la clase (Class<?>)

```java
// Las tres formas
Class<?> c1 = String.class;                      // literal
Class<?> c2 = "Hola".getClass();                 // desde instancia
Class<?> c3 = Class.forName("java.lang.String"); // desde nombre

// Info básica
c1.getName();          // "java.lang.String"
c1.getSimpleName();    // "String"
c1.getPackageName();   // "java.lang"
c1.getSuperclass();    // class java.lang.Object
c1.getInterfaces();    // interfaces que implementa
c1.isInterface();      // false
c1.isEnum();           // false
c1.isRecord();         // false
```

## Inspeccionar métodos

```java
Class<?> clazz = Usuario.class;

// getMethod — público (incluyendo heredados)
Method metodo = clazz.getMethod("getNombre");

// getDeclaredMethod — todos los propios (incluyendo privados)
Method privado = clazz.getDeclaredMethod("metodoPrivado");

// Todos los métodos públicos
Method[] metodos = clazz.getMethods();

// Invocar método
Object instancia = clazz.getDeclaredConstructor().newInstance();
Object resultado = metodo.invoke(instancia);

// Invocar método privado
privado.setAccessible(true);
privado.invoke(instancia);
```

## Inspeccionar campos

```java
// Leer campo privado
Field campo = clazz.getDeclaredField("nombre");
campo.setAccessible(true);
String valor = (String) campo.get(instancia);

// Modificar campo
campo.set(instancia, "Nuevo nombre");

// Todos los campos declarados
Field[] campos = clazz.getDeclaredFields();
for (Field f : campos) {
    System.out.println(f.getName() + " : " + f.getType().getSimpleName());
}
```

## Constructores

```java
// Constructor por defecto
Constructor<?> constructor = clazz.getDeclaredConstructor();
Object obj = constructor.newInstance();

// Constructor con parámetros
Constructor<?> c = clazz.getDeclaredConstructor(String.class, int.class);
Object obj = c.newInstance("Juber", 25);
```

## Leer anotaciones

```java
Method metodo = clazz.getMethod("crearUsuario", Usuario.class);
if (metodo.isAnnotationPresent(Transactional.class)) {
    Transactional ann = metodo.getAnnotation(Transactional.class);
    System.out.println("Propagación: " + ann.propagation());
}

// Anotaciones en campos
for (Field f : clazz.getDeclaredFields()) {
    if (f.isAnnotationPresent(Column.class)) {
        Column col = f.getAnnotation(Column.class);
        System.out.println("Columna: " + col.name());
    }
}
```

## Dynamic Proxy

Crear implementaciones de interfaces en tiempo de ejecución.

```java
public interface Saludo {
    String saludar(String nombre);
}

Saludo proxy = (Saludo) Proxy.newProxyInstance(
    Saludo.class.getClassLoader(),
    new Class[]{Saludo.class},
    (proxyObj, method, args) -> {
        System.out.println("Antes de llamar: " + method.getName());
        String resultado = "Hola, " + args[0];
        System.out.println("Después de llamar");
        return resultado;
    }
);

proxy.saludar("Juber"); // imprime el before/after y devuelve "Hola, Juber"
```

> [!tip] Así es como Spring implementa AOP y los `@Transactional` — con Dynamic Proxies.

## ⚠️ Errores Comunes

> [!warning] `setAccessible(true)` puede fallar con módulos (JPMS)
> En Java 9+ con módulos, acceder a campos privados de otros módulos puede lanzar `InaccessibleObjectException`.

> [!warning] Reflection es costoso
> Evitar en código de alto rendimiento. Cachear los objetos `Method`, `Field` si se usan repetidamente.

## 📌 Notas Rápidas
