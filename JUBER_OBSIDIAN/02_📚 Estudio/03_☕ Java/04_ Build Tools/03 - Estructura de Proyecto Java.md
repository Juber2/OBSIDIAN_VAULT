---
tags:
  - Java
  - Build Tools
Creado: 2026-05-21
Relacionado:
  - Build Tools
  - Spring Boot
Palabras clave:
  - estructura de proyecto, paquetes, convenciones, Spring Boot layout
---

## 🧠 Estructura estándar Maven/Gradle

```
mi-proyecto/
├── src/
│   ├── main/
│   │   ├── java/                    → código fuente
│   │   │   └── com/juber/miapp/
│   │   │       ├── MiAppApplication.java
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       ├── model/ (o entity/)
│   │   │       ├── dto/
│   │   │       ├── config/
│   │   │       ├── exception/
│   │   │       └── util/
│   │   └── resources/
│   │       ├── application.properties (o .yml)
│   │       ├── application-dev.properties
│   │       ├── application-prod.properties
│   │       ├── static/              → recursos estáticos (si aplica)
│   │       └── templates/           → templates Thymeleaf (si aplica)
│   └── test/
│       ├── java/
│       │   └── com/juber/miapp/     → misma estructura que main
│       └── resources/
│           └── application-test.properties
├── pom.xml (Maven) / build.gradle.kts (Gradle)
├── .gitignore
├── Dockerfile
└── README.md
```

## Convención de paquetes

```
com.{empresa}.{proyecto}.{capa}

com.juber.miapp.controller
com.juber.miapp.service
com.juber.miapp.repository
com.juber.miapp.model
com.juber.miapp.dto
com.juber.miapp.config
com.juber.miapp.exception
com.juber.miapp.util
```

## Organización por feature (alternativa moderna)

```
com/juber/miapp/
├── usuario/
│   ├── UsuarioController.java
│   ├── UsuarioService.java
│   ├── UsuarioRepository.java
│   ├── Usuario.java
│   └── UsuarioDTO.java
├── pedido/
│   ├── PedidoController.java
│   ├── PedidoService.java
│   └── ...
└── shared/
    ├── exception/
    └── config/
```

> [!tip] Por capa vs por feature
> **Por capa** (controller/service/repo): común en proyectos pequeños-medianos.
> **Por feature**: mejor para proyectos grandes, facilita identificar toda la lógica de un dominio.

## Convenciones de nombres

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| Clase | PascalCase | `UsuarioController` |
| Interfaz | PascalCase | `UsuarioRepository` |
| Método | camelCase | `buscarPorId` |
| Variable | camelCase | `nombreUsuario` |
| Constante | UPPER_SNAKE_CASE | `MAX_REINTENTOS` |
| Paquete | lowercase | `com.juber.miapp` |
| Archivo | Igual que la clase | `UsuarioController.java` |

## .gitignore estándar para Java

```gitignore
# Build
target/
build/
*.jar
*.war
*.class

# IDE
.idea/
*.iml
.vscode/
.eclipse/
*.project
*.classpath

# OS
.DS_Store
Thumbs.db

# Environment
.env
application-local.properties
application-secrets.yml
```

## 📌 Notas Rápidas
- Organización "por feature" escala mejor en microservicios/equipos grandes — reduce el acoplamiento cognitivo de tener que saltar entre 5 paquetes por cada cambio de una funcionalidad.
- `src/test/java` debe reflejar el mismo paquete que la clase que testea — convención que además permite acceder a métodos `package-private` desde el test.
- `application-{profile}.properties` sobrescribe solo las propiedades definidas ahí — el `application.properties` base sigue aplicando para el resto.
- Nunca commitear `application-secrets.yml`/`.env` con credenciales reales — deben ir al `.gitignore` desde el día uno del proyecto.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

