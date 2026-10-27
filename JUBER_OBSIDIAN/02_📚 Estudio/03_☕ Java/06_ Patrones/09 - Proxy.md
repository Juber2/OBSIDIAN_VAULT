---
tags:
  - Java
  - Patrones
  - Estructural
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
  - Reflection
Palabras clave:
  - proxy, control de acceso, lazy loading, logging, caché, AOP, Dynamic Proxy
---

## 🧠 Concepto

Proporciona un **sustituto** de otro objeto para controlar el acceso a él. El proxy implementa la misma interfaz y añade comportamiento antes/después de delegar al objeto real.

**Tipos:** Virtual (lazy loading), Protection (control de acceso), Remote, Cache, Logging.

## Implementación

```java
public interface ServicioImagen {
    void mostrar();
}

// Objeto real — costoso de crear
public class ImagenReal implements ServicioImagen {
    private final String archivo;

    public ImagenReal(String archivo) {
        this.archivo = archivo;
        cargarDescoEstructura(); // costoso
    }

    private void cargarDescoEstructura() {
        System.out.println("Cargando imagen del disco: " + archivo);
    }

    public void mostrar() {
        System.out.println("Mostrando: " + archivo);
    }
}

// Proxy Virtual — lazy loading
public class ProxyImagen implements ServicioImagen {
    private final String archivo;
    private ImagenReal imagenReal; // null hasta que se necesite

    public ProxyImagen(String archivo) {
        this.archivo = archivo; // no carga nada aún
    }

    public void mostrar() {
        if (imagenReal == null) {
            imagenReal = new ImagenReal(archivo); // lazy: carga solo al usar
        }
        imagenReal.mostrar();
    }
}

// Uso
ServicioImagen imagen = new ProxyImagen("foto.jpg"); // no carga
// ...
imagen.mostrar(); // ahora sí carga y muestra
imagen.mostrar(); // ya está cargada, solo muestra
```

## Proxy de protección

```java
public class ProxyServicioUsuario implements ServicioUsuario {
    private final ServicioUsuario real;
    private final UsuarioActual usuarioActual;

    public ProxyServicioUsuario(ServicioUsuario real, UsuarioActual usuarioActual) {
        this.real = real;
        this.usuarioActual = usuarioActual;
    }

    public void eliminarUsuario(Long id) {
        if (!usuarioActual.tieneRol("ADMIN")) {
            throw new AccesoDenegadoException("Solo admins pueden eliminar usuarios");
        }
        real.eliminarUsuario(id);
    }
}
```

## Dynamic Proxy (Java)

```java
// Proxy dinámico — sin necesidad de implementar clase por clase
ServicioUsuario proxy = (ServicioUsuario) Proxy.newProxyInstance(
    ServicioUsuario.class.getClassLoader(),
    new Class[]{ServicioUsuario.class},
    (proxyObj, method, args) -> {
        System.out.println(">> Llamando: " + method.getName());
        long inicio = System.currentTimeMillis();
        Object resultado = method.invoke(servicioReal, args);
        System.out.println(">> Duración: " + (System.currentTimeMillis() - inicio) + "ms");
        return resultado;
    }
);

// Así funciona @Transactional, @Cacheable, @Async en Spring
```

## Proxy vs Decorator

| | Proxy | Decorator |
|--|-------|-----------|
| Propósito | Controlar acceso al objeto | Añadir comportamiento |
| Creación | Puede crear el objeto real internamente | Recibe el objeto como parámetro |
| Conocimiento | Puede controlar cuándo/si delegar | Siempre delega |

## 📌 Notas Rápidas
- Es el mecanismo interno detrás de casi toda la "magia" de Spring: `@Transactional`, `@Cacheable`, `@Async` funcionan porque Spring genera un proxy dinámico alrededor del bean real.
- Dos formas de proxy en Spring: JDK Dynamic Proxy (requiere que la clase implemente una interfaz) y CGLIB (genera una subclase, funciona sin interfaz) — Spring Boot usa CGLIB por defecto.
- Un error clásico: llamar a un método `@Transactional` desde otro método de la **misma clase** no pasa por el proxy — la transacción no se aplica (self-invocation problem).
- Distinto de Adapter: Proxy mantiene la **misma interfaz** que el objeto real; Adapter traduce entre interfaces **distintas**.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

