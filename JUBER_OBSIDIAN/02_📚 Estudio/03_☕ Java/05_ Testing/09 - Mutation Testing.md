---
tags:
  - Java
  - Testing
  - Calidad
Creado: 2026-09-08
Relacionado:
  - Testing en Java
  - 01 - JUnit 5
Palabras clave:
  - PIT, mutantes, mutation score, cobertura, mutantes equivalentes, incremental
---

## 🧠 Concepto

La cobertura mide **qué líneas se ejecutan**. El *mutation testing* mide **qué errores detectan tus tests**, que es lo que en realidad querías saber.

```java
// ❌ Este test da 100% de cobertura y no prueba absolutamente nada
@Test
void calculaElInteres() {
    calculadora.calcular(new BigDecimal("1000"), TASA, 30);
    // sin un solo assert
}
```

> [!danger] La cobertura se puede falsear; el mutation score no
> Un test sin `assert` ejecuta el código y suma cobertura. Un equipo con la meta de "80% de cobertura" puede alcanzarla escribiendo tests que no verifican nada — y eso pasa más de lo que uno querría.
> El mutation testing rompe el código a propósito y verifica que **algún test se ponga rojo**. Si ninguno se rompe, ese test no estaba probando nada.

## 🧬 Cómo funciona

```
1 · PIT toma tu código compilado
2 · Genera MUTANTES: versiones con un cambio pequeño
     if (a > b)  →  if (a >= b)
     return x;   →  return 0;
     a + b       →  a - b
3 · Ejecuta tus tests contra cada mutante
4 · Si algún test falla → el mutante fue MATADO ✅  (tus tests lo detectan)
   Si todos pasan      → el mutante SOBREVIVIÓ ❌  (nadie lo detecta)
```

```
Mutation score = mutantes matados / mutantes totales
```

## 🚀 Puesta en marcha

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.17.0</version>
    <dependencies>
        <dependency>
            <groupId>org.pitest</groupId>
            <artifactId>pitest-junit5-plugin</artifactId>
            <version>1.2.1</version>
        </dependency>
    </dependencies>
    <configuration>
        <targetClasses>
            <param>com.corebank.dominio.*</param>
            <param>com.corebank.aplicacion.*</param>
        </targetClasses>
        <targetTests>
            <param>com.corebank.*Test</param>
        </targetTests>
        <mutationThreshold>75</mutationThreshold>
        <timestampedReports>false</timestampedReports>
    </configuration>
</plugin>
```

```bash
./mvnw test-compile org.pitest:pitest-maven:mutationCoverage
```

## 🔬 Los mutadores

| Mutador | Transformación | Ejemplo |
|---|---|---|
| **Condicionales de borde** | `<` ↔ `<=`, `>` ↔ `>=` | `saldo > monto` → `saldo >= monto` |
| **Negación de condicional** | `==` ↔ `!=` | `if (a == b)` → `if (a != b)` |
| **Matemáticos** | `+` ↔ `-`, `*` ↔ `/` | `a + b` → `a - b` |
| **Incrementos** | `++` ↔ `--` | |
| **Valores de retorno** | devuelve un valor distinto | `return x` → `return null` |
| **Invocaciones nulas** | elimina la llamada | `log.info(...)` → nada |
| **Constructores** | devuelve `null` | |
| **Constantes** | cambia el literal | `1` → `0` |

> [!tip] El mutador de condicionales de borde es el que más encuentra
> `saldo > monto` versus `saldo >= monto` es la diferencia entre permitir o no una transferencia que deja el saldo exactamente en cero. Si ningún test cubre ese caso límite, el mutante sobrevive — y ese es exactamente el bug que aparece en producción.

## 🔍 Leer el reporte

```
com.corebank.dominio.servicio.PoliticaDeLimites

  12  if (disponible.compareTo(monto) < 0) {
      SOBREVIVIÓ · changed conditional boundary
      → ningún test cubre el caso disponible == monto

  18  return Decision.permitir();
      MATADO · mutated return value

  25  auditoria.registrar(evento);
      SOBREVIVIÓ · removed call
      → ningún test verifica que se audite
```

Cada mutante sobreviviente es una pregunta concreta: **¿qué test falta?**

```java
// El mutante de la línea 12 muere con este test
@Test
void transferir_exactamente_el_saldo_disponible_esta_permitido() {
    crearCuentaCon(cuenta, Dinero.de("500", PEN));
    assertDoesNotThrow(() -> servicio.transferir(cmd(cuenta, destino, "500")));
}

// El de la línea 25 muere con este
@Test
void toda_transferencia_queda_auditada() {
    servicio.transferir(cmd);
    verify(auditoria).registrar(argThat(e -> e.accion().equals("TRANSFERENCIA")));
}
```

> [!tip] Un mutante sobreviviente es un caso de prueba que falta
> No es una métrica abstracta: te está señalando exactamente qué comportamiento no está verificado. Es la forma más concreta de saber qué test escribir.

## ⚖️ Cobertura vs mutation score

| Situación | Cobertura | Mutation score |
|---|---|---|
| Tests sin asserts | 100% | ~0% |
| Tests que solo verifican "no lanza excepción" | 100% | bajo |
| Tests con asserts sobre el resultado | 100% | alto |
| Casos límite cubiertos | 100% | muy alto |

```
Cobertura 90% + mutation score 30%  →  los tests ejecutan pero no verifican
Cobertura 70% + mutation score 85%  →  menos código cubierto, MUCHO mejor probado
```

> [!warning] Un mutation score alto con cobertura baja es mejor que lo inverso
> La cobertura mide superficie; el mutation score mide profundidad. Preferí tests que verifiquen bien lo importante antes que tests que toquen todo sin verificar nada.

## 🎯 Dónde aplicarlo

| Aplicar ✅ | No aplicar ❌ |
|---|---|
| Dominio y reglas de negocio | DTOs y `record` |
| Cálculos financieros | configuración de Spring |
| Validaciones y políticas | mappers generados |
| Máquinas de estado | controladores delgados |
| Cálculo de límites y comisiones | código generado |

```xml
<excludedClasses>
    <param>com.corebank.infraestructura.config.*</param>
    <param>com.corebank.api.dto.*</param>
</excludedClasses>
```

> [!danger] Es costoso: minutos, no segundos
> PIT ejecuta la suite completa (o la parte relevante) **una vez por cada mutante**. Con 500 mutantes y una suite de 10 segundos, son ~80 minutos.
> Por eso se acota a las clases donde importa, y se usa análisis incremental.

## ⚡ Análisis incremental

```xml
<withHistory>true</withHistory>
<historyInputFile>${project.basedir}/.pit-history</historyInputFile>
<historyOutputFile>${project.basedir}/.pit-history</historyOutputFile>
```

Solo analiza lo que cambió desde la última ejecución. La primera vez tarda; las siguientes, segundos.

```xml
<!-- O directamente: solo el código modificado en la rama -->
<features>
    <feature>+GIT(from[main])</feature>
</features>
```

> [!tip] Incremental sobre el diff es lo que lo hace viable en CI
> Analizar solo las clases modificadas respecto de `main` convierte 80 minutos en menos de uno, y sigue cubriendo lo que importa: **el código nuevo**.

## 👻 Mutantes equivalentes

Algunos mutantes son imposibles de matar porque **no cambian el comportamiento**.

```java
// El mutante cambia >= por >, pero el resultado es idéntico
for (int i = 0; i < lista.size(); i++) { ... }
// mutado a i != lista.size() → mismo comportamiento

// Optimización que no altera el resultado observable
if (cache.contains(k)) return cache.get(k);      // quitar esto: más lento, igual resultado
```

> [!warning] Por eso el objetivo nunca es 100%
> Los mutantes equivalentes existen y detectarlos automáticamente es un problema indecidible. Un mutation score del 100% no es alcanzable ni deseable.
> **Rangos razonables**: 70-85% en dominio y aplicación; por debajo de 60% hay huecos reales.

```xml
<!-- Se pueden ignorar líneas concretas -->
<avoidCallsTo>
    <avoidCallsTo>org.slf4j</avoidCallsTo>       <!-- los logs no se testean -->
</avoidCallsTo>
```

## 📊 En el pipeline

```yaml
- name: Mutation testing (solo el código nuevo)
  run: |
    ./mvnw test-compile org.pitest:pitest-maven:mutationCoverage \
      -DwithHistory=true \
      -Dfeatures="+GIT(from[origin/main])" \
      -DmutationThreshold=75
```

> [!tip] Umbral sobre el código nuevo, no sobre todo el proyecto
> Exigir 75% global en un proyecto existente es inviable. Exigir 75% **en lo que agregás** es razonable y mejora el proyecto de forma sostenida. Es la misma lógica del `FreezingArchRule` de [[07 - ArchUnit]].

## 🧪 Un ejemplo completo

```java
public class PoliticaDeLimites {
    public void verificar(Cuenta cuenta, Dinero monto) {
        Dinero consumido = consumoDiario(cuenta);
        Dinero limite = cuenta.limiteDiario();

        if (consumido.mas(monto).compareTo(limite) > 0) {
            throw new LimiteExcedidoException(limite, consumido);
        }
    }
}
```

```java
// Este test da 100% de cobertura y deja mutantes vivos
@Test
void rechaza_si_excede_el_limite() {
    assertThrows(LimiteExcedidoException.class,
        () -> politica.verificar(cuenta, Dinero.de("20000", PEN)));
}
```

```
Mutantes sobrevivientes:
  > 0  →  >= 0    SOBREVIVIÓ · falta el caso "exactamente el límite"
  mas  →  menos   SOBREVIVIÓ · ningún test verifica que SUME el consumo previo
```

```java
// Estos dos los matan
@Test
void permite_exactamente_el_limite() {
    conConsumoDe("10000");
    assertDoesNotThrow(() -> politica.verificar(cuenta, Dinero.de("5000", PEN)));
    // límite 15.000, consumido 10.000, monto 5.000 → exactamente el límite
}

@Test
void el_consumo_previo_se_suma_al_monto_nuevo() {
    conConsumoDe("14000");
    assertThrows(LimiteExcedidoException.class,
        () -> politica.verificar(cuenta, Dinero.de("2000", PEN)));
    // sin sumar el consumo previo, 2.000 pasaría
}
```

> [!tip] El segundo test es el que vale
> Verifica que la política **acumula** el consumo, no solo que rechaza montos grandes. Es un comportamiento crítico que la cobertura declaraba probado y no lo estaba.

## ⚠️ Errores Comunes

> [!warning] Los que hacen que se abandone
> - **Aplicarlo a todo el proyecto**: horas de ejecución, se desactiva.
> - **Sin análisis incremental**: inviable en CI.
> - **Perseguir el 100%**: los mutantes equivalentes lo impiden.
> - **Incluir DTOs y configuración**: ruido sin valor.
> - **No excluir logging**: mutantes que nadie va a matar.
> - **Umbral global en un proyecto existente**: bloquea desde el día uno.
> - **Ignorar los sobrevivientes**: la métrica sin acción no sirve de nada.

## 📌 Notas Rápidas
- La frase que resume el valor: **la cobertura mide qué se ejecuta; el mutation score mide qué se detecta**. Mencionar esa distinción en una entrevista demuestra criterio sobre calidad de tests.
- Cada mutante sobreviviente es un caso de prueba faltante, señalado con línea y motivo. Es la herramienta más concreta que existe para saber qué test escribir.
- El mutador de condicionales de borde (`>` ↔ `>=`) es el que más bugs reales encuentra, porque los casos límite son justamente los que nadie prueba.
- Se aplica en [[Fase 4 - Hexagonal y Testing]] con umbral del 75% en dominio y aplicación.

## 🔗 Relaciones
- **Módulo**: [[Testing en Java]]
- **Anterior**: [[08 - Contract Testing]]
- **Siguiente**: [[10 - Pruebas de Carga]]
- **Se conecta con**: [[01 - JUnit 5]], [[03 - TDD]], [[07 - ArchUnit]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

