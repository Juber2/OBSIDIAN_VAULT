---
tags:
  - Java
  - Testing
  - Arquitectura
Creado: 2026-09-08
Relacionado:
  - Testing en Java
  - 03 - Arquitectura Hexagonal
Palabras clave:
  - ArchUnit, reglas de arquitectura, capas, ciclos, convenciones, congelar violaciones
---

## 🧠 Concepto

Las reglas de arquitectura suelen vivir en un documento que nadie relee. **ArchUnit las convierte en tests**: si alguien viola la separación de capas, el build se pone rojo.

```
Convención en el README   →  alguien la olvida, nadie lo nota
Regla en un test           →  el pipeline falla, no se puede ignorar
```

## 🚀 Puesta en marcha

```xml
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@AnalyzeClasses(packages = "com.corebank",
                importOptions = ImportOption.DoNotIncludeTests.class)
class ReglasDeArquitectura {

    @ArchTest
    static final ArchRule el_dominio_no_conoce_frameworks =
        noClasses().that().resideInAPackage("..dominio..")
            .should().dependOnClassesThat()
            .resideInAnyPackage("org.springframework..",
                                "jakarta.persistence..",
                                "com.fasterxml.jackson..");
}
```

ArchUnit lee el bytecode compilado y verifica las reglas. No necesita levantar Spring ni base de datos: corre en milisegundos.

## 🏛️ Reglas de capas

```java
@ArchTest
static final ArchRule capas_respetadas = layeredArchitecture()
    .consideringOnlyDependenciesInLayers()
    .layer("Entrada").definedBy("..infraestructura.entrada..")
    .layer("Aplicacion").definedBy("..aplicacion..")
    .layer("Dominio").definedBy("..dominio..")
    .layer("Salida").definedBy("..infraestructura.salida..")

    .whereLayer("Entrada").mayNotBeAccessedByAnyLayer()
    .whereLayer("Aplicacion").mayOnlyBeAccessedByLayers("Entrada")
    .whereLayer("Dominio").mayOnlyBeAccessedByLayers("Aplicacion", "Entrada", "Salida")
    .whereLayer("Salida").mayNotBeAccessedByAnyLayer();
```

```java
@ArchTest
static final ArchRule el_dominio_no_conoce_la_infraestructura =
    noClasses().that().resideInAPackage("..dominio..")
        .should().dependOnClassesThat().resideInAPackage("..infraestructura..");

@ArchTest
static final ArchRule los_controladores_no_tocan_repositorios =
    noClasses().that().resideInAPackage("..entrada.rest..")
        .should().dependOnClassesThat().resideInAPackage("..salida.persistencia..");

@ArchTest
static final ArchRule las_entidades_jpa_no_salen_de_persistencia =
    classes().that().areAnnotatedWith(Entity.class)
        .should().resideInAPackage("..salida.persistencia..");
```

> [!tip] Estas cuatro reglas son las que hacen que un refactor a hexagonal no se degrade
> Después de invertir días en separar el dominio de la infraestructura, alcanza con que alguien agregue un `import org.springframework` "por comodidad" para empezar a deshacerlo. La regla lo impide desde el primer commit — ver [[Fase 4 - Hexagonal y Testing]].

## 🔄 Ciclos

```java
@ArchTest
static final ArchRule sin_ciclos_entre_paquetes =
    slices().matching("com.corebank.(*)..").should().beFreeOfCycles();
```

> [!danger] Un ciclo entre paquetes es un problema estructural
> Si `pedidos` depende de `facturacion` y `facturacion` depende de `pedidos`, no se pueden entender, testear ni desplegar por separado. Y los ciclos crecen: una vez que existe uno, agregar dependencias en ambas direcciones deja de parecer un problema.
> Es la regla que más seguido encuentra deuda real en un proyecto existente.

## 📛 Convenciones de nombres

```java
@ArchTest
static final ArchRule los_controladores_terminan_en_Controller =
    classes().that().areAnnotatedWith(RestController.class)
        .should().haveSimpleNameEndingWith("Controller");

@ArchTest
static final ArchRule los_puertos_son_interfaces =
    classes().that().resideInAPackage("..aplicacion.puerto..")
        .should().beInterfaces();

@ArchTest
static final ArchRule los_casos_de_uso_terminan_en_UseCase =
    classes().that().resideInAPackage("..aplicacion.puerto.entrada..")
        .should().haveSimpleNameEndingWith("UseCase");
```

## 💰 Reglas de dominio

Acá es donde ArchUnit deja de ser una herramienta genérica y se vuelve específica de tu problema.

```java
@ArchTest
static final ArchRule ningun_importe_usa_double =
    noFields().should().haveRawType(double.class)
        .orShould().haveRawType(Double.class)
        .orShould().haveRawType(float.class)
        .orShould().haveRawType(Float.class)
        .because("todo importe monetario debe ser BigDecimal");
```

> [!tip] Es la regla más valiosa del laboratorio
> Convierte la regla número uno del dominio bancario —nunca `double` para dinero— en algo que **falla el build**. Escrita en el README se olvida; escrita como test, no se puede violar sin que alguien lo vea.

```java
@ArchTest
static final ArchRule sin_System_out =
    noClasses().should().callMethod(System.class, "currentTimeMillis")
        .because("para medir duraciones va System.nanoTime()");   // ver [[11 - Relojes y Causalidad]]

@ArchTest
static final ArchRule los_servicios_no_usan_inyeccion_por_campo =
    noFields().that().areDeclaredInClassesThat().areAnnotatedWith(Service.class)
        .should().beAnnotatedWith(Autowired.class)
        .because("inyección por constructor: dependencias explícitas y testeables");

@ArchTest
static final ArchRule las_transacciones_no_van_en_metodos_privados =
    methods().that().areAnnotatedWith(Transactional.class)
        .should().bePublic()
        .because("el proxy de Spring no intercepta métodos privados");
```

> [!warning] La última regla atrapa un bug real y silencioso
> `@Transactional` en un método privado (o llamado internamente) **no hace nada**: el proxy no lo intercepta. No hay error, no hay aviso, simplemente no hay transacción. Es de los bugs más difíciles de detectar en revisión de código y de los más fáciles de detectar con una regla.

## 🎯 Reglas personalizadas

```java
@ArchTest
static final ArchRule las_entidades_no_se_devuelven_desde_controladores =
    noMethods().that().areDeclaredInClassesThat().areAnnotatedWith(RestController.class)
        .should(devolverEntidadJpa());

private static ArchCondition<JavaMethod> devolverEntidadJpa() {
    return new ArchCondition<>("no devolver entidades JPA") {
        @Override
        public void check(JavaMethod metodo, ConditionEvents eventos) {
            JavaClass retorno = metodo.getRawReturnType();
            if (retorno.isAnnotatedWith(Entity.class)) {
                eventos.add(SimpleConditionEvent.violated(metodo,
                    "%s devuelve la entidad %s".formatted(metodo.getFullName(), retorno.getName())));
            }
        }
    };
}
```

## 🧊 Congelar violaciones existentes

El problema al adoptar ArchUnit en un proyecto con años: la primera ejecución encuentra 400 violaciones y nadie va a arreglarlas hoy.

```java
@ArchTest
static final ArchRule sin_ciclos = FreezingArchRule.freeze(
    slices().matching("com.corebank.(*)..").should().beFreeOfCycles());
```

```
Primera ejecución  → guarda las 400 violaciones actuales en un archivo
Ejecuciones futuras → falla SOLO si aparece una violación NUEVA
Si arreglás una     → se elimina del archivo automáticamente
```

> [!tip] Congelar es lo que hace viable adoptarlo en un proyecto existente
> Detiene la degradación sin exigir arreglar todo primero. El archivo de violaciones va al repositorio y solo puede achicarse — es una barrera de trinquete.
> Es la única forma realista de introducir estas reglas en código heredado.

## ⚡ En el pipeline

```java
@AnalyzeClasses(packages = "com.corebank")
class ReglasDeArquitectura {
    // Todas las @ArchTest se ejecutan como tests normales de JUnit 5
}
```

```yaml
- name: Reglas de arquitectura
  run: ./mvnw test -Dtest=ReglasDeArquitectura
```

> [!tip] Corren en milisegundos: ponelas temprano en el pipeline
> No levantan Spring, no necesitan base de datos, no compilan nada extra. Ubicarlas junto a los tests unitarios da retroalimentación en segundos, antes de esperar los tests de integración.

## 📊 Qué reglas poner

| Prioridad | Regla | Por qué |
|---|---|---|
| **Alta** | dominio sin frameworks | protege el núcleo |
| **Alta** | sin ciclos entre paquetes | deuda estructural |
| **Alta** | reglas de dominio (sin `double` para dinero) | previenen bugs de negocio |
| Media | entidades JPA acotadas a persistencia | evita filtrar el modelo |
| Media | inyección por constructor | testabilidad |
| Media | `@Transactional` público | bug silencioso |
| Baja | convenciones de nombres | consistencia |

> [!warning] Empezá con pocas reglas y de alto valor
> Cincuenta reglas de nomenclatura generan ruido y terminan desactivándose. Cinco reglas que protegen decisiones arquitectónicas importantes se respetan.

## 🧪 Cómo se ve un fallo

```
java.lang.AssertionError: Architecture Violation [Priority: MEDIUM] -
Rule 'no classes that reside in a package '..dominio..' should depend on classes that
reside in any package ['org.springframework..']' was violated (2 times):

Class <com.corebank.dominio.modelo.Cuenta> depends on
  <org.springframework.util.Assert> in (Cuenta.java:34)
Class <com.corebank.dominio.servicio.PoliticaDeLimites> depends on
  <org.springframework.stereotype.Component> in (PoliticaDeLimites.java:12)
```

> [!tip] El mensaje dice clase, línea y motivo
> A diferencia de una revisión de código donde alguien tiene que notarlo, acá el error señala exactamente dónde está el problema. Eso hace que se arregle en vez de discutirse.

## ⚠️ Errores Comunes

> [!warning] Los que hacen que se abandone
> - **Demasiadas reglas al empezar**: ruido, y se terminan desactivando.
> - **Reglas de nomenclatura sin valor arquitectónico** mezcladas con las importantes.
> - **No congelar** en un proyecto existente: 400 fallos el primer día.
> - **Reglas sin `because(...)`**: quien las rompe no entiende por qué existen.
> - **Incluir los tests en el análisis**: suelen violar reglas legítimamente.
> - **Ponerlas al final del pipeline**: la retroalimentación llega tarde.

## 📌 Notas Rápidas
- La regla que más impacto tiene en un proyecto financiero es `ningun_importe_usa_double`. Convierte una convención frágil en una garantía del build.
- El `because(...)` no es decorativo: aparece en el mensaje de error y le explica a quien rompió la regla por qué existe. Sin eso, la reacción típica es desactivarla.
- Congelar violaciones (`FreezingArchRule`) es lo que permite introducir ArchUnit en código heredado sin bloquear al equipo. Es la funcionalidad menos conocida y la más útil en la práctica.
- Se implementa en [[Fase 4 - Hexagonal y Testing]].

## 🔗 Relaciones
- **Módulo**: [[Testing en Java]]
- **Siguiente**: [[08 - Contract Testing]]
- **Se conecta con**: [[03 - Arquitectura Hexagonal]], [[02 - Clean Architecture en Java]], [[01 - Dinero en Software]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

