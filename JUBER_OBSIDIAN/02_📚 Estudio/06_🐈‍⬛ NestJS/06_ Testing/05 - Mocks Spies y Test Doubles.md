---
tags:
  - NestJS
  - TypeScript
  - Nest-Testing
Creado: 2026-07-27
Relacionado:
  - Testing NestJS
  - NESTJS
---

# 05 - Mocks Spies y Test Doubles

## Que vas a aprender

En este capitulo vas a profundizar en los distintos tipos de **test doubles** (stubs, spies, mocks, fakes) disponibles con Jest, y cuando usar cada uno para escribir tests claros y mantenibles.

## Explicacion

No todos los "dobles de prueba" son iguales. Un **stub** devuelve respuestas predefinidas sin verificar nada mas (`jest.fn().mockReturnValue(x)`). Un **spy** envuelve una funcion real y registra como fue llamada, permitiendo verificar interacciones sin reemplazar el comportamiento (`jest.spyOn`). Un **mock** completo reemplaza toda la dependencia con una version controlada. Un **fake** es una implementacion simplificada pero funcional (por ejemplo, un repositorio en memoria en vez de uno que hable con una base de datos real).

Elegir el tipo correcto importa: usar mocks para todo, incluso cuando un simple stub alcanza, agrega complejidad innecesaria a los tests y los acopla mas de lo necesario a detalles de implementacion.

## Modelo mental

Pensa estos test doubles como distintos niveles de actor de utileria: un stub es un cartel con una respuesta escrita, un spy es una camara oculta sobre algo real, un mock es un actor completo actuando el papel, y un fake es un doble de riesgo que realmente hace el truco pero de forma simplificada.

## Ejemplo base

```typescript
// Stub: devuelve un valor fijo
const stub = jest.fn().mockReturnValue({ id: '1' });

// Spy: observa una llamada real sin cambiar su comportamiento
const spy = jest.spyOn(mailerService, 'send');
await usersService.register(dto);
expect(spy).toHaveBeenCalledWith(dto.email, expect.any(String));

// Fake: repositorio en memoria para tests rapidos sin base de datos real
class InMemoryUsersRepository {
  private users: User[] = [];
  save(user: User) { this.users.push(user); return Promise.resolve(user); }
  findOne(id: string) { return Promise.resolve(this.users.find((u) => u.id === id)); }
}
```

## Como se usa en un proyecto real

1. Se usan stubs simples cuando solo importa el valor de retorno, no la interaccion.
2. Se usan spies cuando se quiere verificar que algo fue llamado, sin perder el comportamiento real de la dependencia.
3. Se usan fakes (implementaciones en memoria) para acelerar tests que de otro modo necesitarian infraestructura real, manteniendo el comportamiento realista.
4. Se documentan las convenciones de test doubles del equipo para evitar mezclar estilos de forma inconsistente entre archivos.
5. Se resetean mocks y spies entre tests (`jest.restoreAllMocks()`) para evitar filtraciones de estado entre casos.

## Criterio junior

Un junior debe poder crear un stub simple con `jest.fn()` y entender la diferencia basica entre un stub y un mock completo.

## Criterio mid

Un mid debe elegir el tipo de test double correcto segun lo que realmente necesita verificar el test, y usar `jest.spyOn` para verificar interacciones sin perder comportamiento real.

## Criterio senior

Un senior debe establecer convenciones de testing para el equipo sobre cuando usar cada tipo de test double, evitando tests sobre-mockeados que se vuelven fragiles ante refactors legitimos.

## Errores comunes

- Usar mocks completos para todo, incluso cuando un stub simple alcanza, aumentando la complejidad de mantenimiento de los tests.
- No resetear spies entre tests, generando falsos positivos o negativos por estado acumulado.
- Confundir verificar comportamiento (que devuelve) con verificar implementacion (como lo hace por dentro), acoplando el test a detalles que deberian poder cambiar libremente.
- Crear fakes tan simplificados que no representan el comportamiento real, dando falsa confianza.

## Practica

- [ ] Crea un stub simple con `jest.fn().mockReturnValue()`.
- [ ] Usa `jest.spyOn` para verificar que un metodo de una dependencia real fue llamado.
- [ ] Implementa un fake en memoria de un repositorio para acelerar un test que antes usaba base de datos real.
- [ ] Revisa un test existente sobre-mockeado y simplificalo al tipo de double minimo necesario.
- [ ] Anota una guia propia de "cuando usar cada tipo de test double" para tu equipo.

## Preguntas de repaso

- Que diferencia hay entre un stub y un spy?
- Cuando conviene usar un fake en memoria en vez de mockear cada metodo individualmente?
- Que riesgo tiene sobre-mockear un test?
- Por que es importante resetear mocks y spies entre tests?
