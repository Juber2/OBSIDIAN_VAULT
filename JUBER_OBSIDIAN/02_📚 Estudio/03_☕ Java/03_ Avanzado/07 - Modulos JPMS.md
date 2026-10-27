---
tags:
  - Java
  - Avanzado
Creado: 2026-05-21
Relacionado:
  - Java Avanzado
Palabras clave:
  - JPMS, modules, module-info.java, requires, exports, Java 9
---

## 🧠 Concepto

El **Java Platform Module System (JPMS)**, introducido en Java 9, permite dividir la aplicación en módulos con dependencias explícitas y encapsulamiento fuerte.

## Estructura básica

```
mi-proyecto/
├── src/
│   ├── com.miapp.core/
│   │   ├── module-info.java
│   │   └── com/miapp/core/Servicio.java
│   └── com.miapp.api/
│       ├── module-info.java
│       └── com/miapp/api/Controlador.java
```

## module-info.java

```java
// com.miapp.core/module-info.java
module com.miapp.core {
    // exportar paquetes para otros módulos
    exports com.miapp.core.service;
    exports com.miapp.core.model;

    // paquete interno — no exportado
    // com.miapp.core.internal — oculto

    // declarar dependencias
    requires java.sql;
    requires com.google.gson;

    // dependencia transitiva (quien me use, también la tendrá)
    requires transitive java.logging;

    // permitir reflection de Spring en mis paquetes
    opens com.miapp.core.model to spring.core;
}
```

```java
// com.miapp.api/module-info.java
module com.miapp.api {
    requires com.miapp.core;     // depende del módulo core
    requires spring.webmvc;

    exports com.miapp.api.controller;
}
```

## Palabras clave

| Keyword | Descripción |
|---------|-------------|
| `module` | Declara el módulo |
| `requires` | Dependencia de otro módulo |
| `requires transitive` | Dependencia transitiva |
| `requires static` | Dependencia solo en compilación |
| `exports` | Expone un paquete a todos |
| `exports ... to` | Expone un paquete solo a módulos específicos |
| `opens` | Permite Reflection en el paquete (en runtime) |
| `opens ... to` | Permite Reflection solo a módulos específicos |
| `uses` | Declara uso de un ServiceLoader |
| `provides ... with` | Proporciona implementación de un servicio |

## ServiceLoader (Service Provider Interface)

```java
// En module-info.java del módulo proveedor
provides com.miapp.api.Notificador with com.miapp.email.EmailNotificador;

// En module-info.java del módulo consumidor
uses com.miapp.api.Notificador;

// En código
ServiceLoader<Notificador> loader = ServiceLoader.load(Notificador.class);
loader.forEach(n -> n.notificar("mensaje"));
```

## ¿Cuándo usar JPMS?

En la mayoría de proyectos Spring Boot o empresariales **no es necesario** configurar JPMS explícitamente — Spring Boot gestiona el classpath automáticamente. JPMS es relevante principalmente en:

- Librerías que se distribuyen a terceros
- Aplicaciones que se compilan con `jlink` (imagen mínima de la JVM)
- Proyectos que requieren encapsulamiento muy estricto

## ⚠️ Errores Comunes

> [!warning] Reflection bloqueada por defecto
> Con módulos, Spring e Hibernate no pueden acceder a campos privados sin `opens`. Esto causa `InaccessibleObjectException` en frameworks que usan Reflection.

## 📌 Notas Rápidas
- JPMS (Java Platform Module System, Java 9+) agrega una capa de encapsulamiento fuerte por encima de los packages — un módulo declara explícitamente qué expone (`exports`) y qué necesita (`requires`).
- Se declara en un archivo `module-info.java` en la raíz del código fuente.
- La mayoría de proyectos backend con Spring/frameworks siguen usando el "classpath clásico" (unnamed module) en vez de módulos explícitos, porque muchas librerías del ecosistema aún no son 100% module-friendly — es más un tema de conocimiento teórico/entrevista que de uso diario.
- `opens` (a diferencia de `exports`) permite acceso por Reflection en runtime a paquetes internos — necesario para que frameworks como Hibernate funcionen con módulos.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

