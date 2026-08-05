---
tags:
  - NestJS
  - TypeScript
  - Nest-Senior
Creado: 2026-07-27
Relacionado:
  - Senior NestJS
  - NESTJS
---

# 04 - Mantenimiento Legacy

## Que vas a aprender

En este capitulo vas a desarrollar el criterio para trabajar con proyectos NestJS **legacy**: como abordar codigo sin tests, migraciones de versiones mayores de Nest, y refactors seguros e incrementales sin reescribir todo desde cero.

## Explicacion

La tentacion frente a codigo legacy dificil de mantener es proponer una reescritura completa, pero eso rara vez es la decision correcta: es costoso, arriesgado, y suele subestimar la cantidad de logica de negocio invisible acumulada en el sistema viejo. El enfoque senior es el **refactor incremental**: agregar tests de caracterizacion (que capturan el comportamiento actual, aunque no sea "correcto" en abstracto) antes de tocar codigo sin cobertura, y refactorizar en pasos pequeños y seguros, verificables en cada paso.

Las migraciones de versiones mayores de Nest (o de sus dependencias clave como TypeORM) tambien deben tratarse con este mismo criterio: leer el changelog de breaking changes, migrar de a un paso, y apoyarse en la suite de tests existente (o construida primero) para verificar que nada se rompio.

## Modelo mental

Pensa el codigo legacy como un edificio ocupado que hay que remodelar sin desalojar a los inquilinos: no se puede demoler todo y reconstruir (el negocio sigue funcionando), asi que se trabaja habitacion por habitacion, verificando que cada cambio no afecte al resto del edificio antes de seguir.

## Ejemplo base

```typescript
// Test de caracterizacion: documenta el comportamiento ACTUAL antes de refactorizar,
// aunque ese comportamiento tenga bugs conocidos que se corregiran despues por separado
describe('PricingService (caracterizacion)', () => {
  it('aplica el descuento actual tal como esta implementado hoy', () => {
    // Se documenta el comportamiento real, no el "deberia ser"
    expect(pricingService.calculate(100, 'VIP')).toBe(85); // asi funciona hoy, para bien o mal
  });
});
```

## Como se usa en un proyecto real

1. Antes de refactorizar codigo legacy sin tests, se agregan tests de caracterizacion que documentan el comportamiento actual.
2. Los refactors se hacen en pasos pequeños, cada uno verificado por la suite de tests, nunca en un unico cambio gigante y arriesgado.
3. Las migraciones de version mayor se planifican leyendo el changelog completo de breaking changes antes de empezar, no a prueba y error.
4. Se prioriza refactorizar las partes del sistema que mas cambian o mas bugs generan, no todo el legacy por igual.
5. Se comunica a negocio el costo real de mantener deuda tecnica critica, para poder priorizar tiempo de refactor junto con nuevas features.

## Criterio junior

Un junior debe poder trabajar dentro de codigo legacy siguiendo patrones ya establecidos, sin introducir cambios riesgosos sin supervision.

## Criterio mid

Un mid debe escribir tests de caracterizacion antes de refactorizar codigo sin cobertura, y ejecutar refactors incrementales verificables paso a paso.

## Criterio senior

Un senior debe priorizar que partes del legacy refactorizar primero segun impacto real, liderar migraciones de versiones mayores con un plan claro, y comunicar a negocio el costo y riesgo de mantener deuda tecnica critica sin atender.

## Errores comunes

- Proponer una reescritura completa como primera opcion frente a codigo legacy dificil, subestimando el costo y riesgo real.
- Refactorizar codigo sin tests existentes sin primero agregar tests de caracterizacion, arriesgando romper comportamiento no documentado.
- Migrar una version mayor sin leer el changelog de breaking changes, descubriendo problemas a los golpes.
- Refactorizar por gusto personal en vez de priorizar segun impacto real en bugs o velocidad de desarrollo.

## Practica

- [ ] Identifica una parte de codigo legacy sin tests y escribe tests de caracterizacion antes de tocarla.
- [ ] Planifica un refactor incremental en pasos pequeños y verificables para esa parte del codigo.
- [ ] Lee el changelog de una version mayor reciente de NestJS y lista los breaking changes relevantes.
- [ ] Prioriza que parte de un sistema legacy que conozcas ameritaria refactor primero, con justificacion.
- [ ] Anota como comunicarias a negocio el costo de no atender una deuda tecnica critica.

## Preguntas de repaso

- Por que una reescritura completa rara vez es la mejor primera opcion frente a codigo legacy?
- Que es un test de caracterizacion y quE problema resuelve antes de refactorizar?
- Que pasos seguirias antes de iniciar una migracion de version mayor de NestJS?
- Como priorizarias que parte de un sistema legacy refactorizar primero?
