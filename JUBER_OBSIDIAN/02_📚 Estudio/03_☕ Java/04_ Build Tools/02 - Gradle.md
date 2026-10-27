---
tags:
  - Java
  - Build Tools
Creado: 2026-05-21
Relacionado:
  - Build Tools
  - Maven
Palabras clave:
  - Gradle, build.gradle, Groovy, Kotlin DSL, tasks, plugins
---

## 🧠 Concepto

**Gradle** es más moderno que Maven: más rápido (builds incrementales y caché), más flexible, con DSL en Groovy o Kotlin.

## build.gradle (Kotlin DSL — recomendado)

```kotlin
// build.gradle.kts
plugins {
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
    kotlin("jvm") version "1.9.20"            // si usas Kotlin
    // o
    id("java")                                 // si usas Java
}

group = "com.juber"
version = "1.0.0"

java {
    sourceCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    runtimeOnly("org.postgresql:postgresql")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

## Configuraciones de dependencias

| Configuración | Equivalente Maven | Descripción |
|--------------|------------------|-------------|
| `implementation` | `compile` | Dependencia de compilación y runtime |
| `api` | `compile` | Como implementation, pero expuesta a consumidores |
| `runtimeOnly` | `runtime` | Solo en runtime |
| `compileOnly` | `provided` | Solo en compilación |
| `testImplementation` | `test` | Solo en tests |
| `annotationProcessor` | `compile + apt` | Procesadores de anotaciones |

## Comandos esenciales

```bash
./gradlew build               # compilar + test + empaquetar
./gradlew test                # solo tests
./gradlew bootRun             # ejecutar Spring Boot
./gradlew clean build         # limpiar + build completo
./gradlew dependencies        # ver dependencias
./gradlew tasks               # listar todas las tareas disponibles
./gradlew bootJar             # generar JAR ejecutable
./gradlew -x test build       # build sin tests
```

## Tasks personalizadas

```kotlin
tasks.register("saludar") {
    doLast {
        println("Hola desde Gradle!")
    }
}

// Ejecutar: ./gradlew saludar
```

## Gradle Wrapper

```bash
# Siempre usar el wrapper (garantiza la misma versión de Gradle)
./gradlew ...   # Linux/Mac
gradlew.bat ... # Windows
```

## Maven vs Gradle

| | Maven | Gradle |
|--|-------|--------|
| Lenguaje config | XML | Groovy / Kotlin DSL |
| Velocidad | Moderada | Más rápido (incremental + caché) |
| Flexibilidad | Convención estricta | Muy flexible |
| Curva aprendizaje | Baja | Media |
| Ecosistema | Más maduro | Creciendo rápido |
| Android | No | Estándar oficial |
| Empresas | Muy común | Creciendo |

## 📌 Notas Rápidas
- `implementation` vs `api`: con `implementation`, si tu módulo depende de una librería, los módulos que dependen de vos NO ven esa librería en su classpath — mejor encapsulamiento que Maven por defecto.
- El caché de builds de Gradle (incremental + build cache) es la razón principal de que sea más rápido que Maven en proyectos grandes, sobre todo en CI con cache compartido entre builds.
- Siempre commitear el `gradlew`/`gradlew.bat` y la carpeta `gradle/wrapper/` al repo — así cualquiera clona y compila sin instalar Gradle manualmente.
- Requisito indispensable de este puesto menciona "Maven y Gradle" — en la práctica un equipo usa uno u otro por proyecto, rara vez ambos a la vez; lo importante es entender los conceptos que comparten (dependencias, scopes/configurations, ciclo de vida).

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

