---
tags:
  - Java
  - Build Tools
Creado: 2026-05-21
Relacionado:
  - Build Tools
Palabras clave:
  - Maven, pom.xml, dependencias, ciclo de vida, plugins, repositorio
---

## 🧠 Concepto

**Maven** es la herramienta de build más usada en el ecosistema Java empresarial. Gestiona dependencias, compila, testea y empaqueta proyectos.

## Estructura `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
             http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!-- Identidad del proyecto -->
    <groupId>com.juber</groupId>
    <artifactId>mi-proyecto</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <!-- Parent (Spring Boot) -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <!-- Propiedades -->
    <properties>
        <java.version>21</java.version>
        <mapstruct.version>1.5.5.Final</mapstruct.version>
    </properties>

    <!-- Dependencias -->
    <dependencies>
        <!-- Scope compile (default) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Scope test — solo en tests -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- Scope provided — el servidor la provee -->
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>

        <!-- Scope runtime — solo en ejecución, no en compilación -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
    </dependencies>

    <!-- Plugins de build -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Ciclo de vida

```
validate → compile → test → package → verify → install → deploy
```

| Fase | Descripción |
|------|-------------|
| `validate` | Verifica que el proyecto está correcto |
| `compile` | Compila el código fuente |
| `test` | Ejecuta los tests unitarios |
| `package` | Empaqueta (JAR/WAR) |
| `verify` | Tests de integración |
| `install` | Instala en repositorio local (~/.m2) |
| `deploy` | Sube al repositorio remoto |

## Comandos esenciales

```bash
mvn compile                    # compilar
mvn test                       # ejecutar tests
mvn package                    # compilar + test + empaquetar JAR
mvn package -DskipTests        # empaquetar sin tests
mvn spring-boot:run            # ejecutar app Spring Boot
mvn install                    # instalar en repo local
mvn clean                      # limpiar target/
mvn clean package              # limpiar + empaquetar (lo más común)
mvn dependency:tree            # ver árbol de dependencias
mvn dependency:resolve         # descargar dependencias
mvn versions:display-dependency-updates  # ver actualizaciones disponibles
```

## Perfiles (profiles)

```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
        </properties>
    </profile>
</profiles>
```

```bash
mvn package -P prod  # activar perfil prod
```

## ⚠️ Errores Comunes

> [!warning] Conflict de dependencias
> Usar `mvn dependency:tree` para detectar versiones conflictivas. Resolver con `<exclusions>` o `<dependencyManagement>`.

## 📌 Notas Rápidas
