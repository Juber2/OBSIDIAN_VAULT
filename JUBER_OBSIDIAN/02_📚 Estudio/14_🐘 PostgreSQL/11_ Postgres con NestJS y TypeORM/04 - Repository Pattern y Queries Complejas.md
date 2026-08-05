---
tags:
  - PostgreSQL
  - NestJS
  - Postgres-NestJS
Creado: 2026-07-29
Relacionado:
  - Postgres con NestJS y TypeORM
  - POSTGRESQL
---

# 04 - Repository Pattern y Queries Complejas

## Que vas a aprender

En este capitulo vas a aprender a construir consultas complejas con el **QueryBuilder** de TypeORM aprovechando features de Postgres, y a decidir cuando usar SQL crudo directamente dentro de un repositorio custom cuando el QueryBuilder se vuelve mas complicado que la alternativa.

## Explicacion

El `QueryBuilder` de TypeORM permite construir consultas SQL complejas de forma programatica (joins, subconsultas, agregaciones, ver capitulos de JOINs y Agregacion de este libro), pero para aprovechar features muy especificas de Postgres (operadores JSONB como `@>`, Full Text Search con `@@`, funciones de array como `unnest()`), a veces es mas simple y legible usar `.query()` con SQL crudo parametrizado directamente, en vez de forzar esa sintaxis especifica dentro de la API del QueryBuilder.

Un **repositorio custom** (extendiendo el repositorio base de TypeORM, ya visto conceptualmente en el capitulo de Entidades y Repositorios del libro de NestJS) es el lugar apropiado para centralizar estas queries complejas reutilizables, manteniendo el servicio de negocio limpio y enfocado en logica de aplicacion, no en detalles de SQL.

## Modelo mental

Pensa el QueryBuilder como un traductor automatico de un lenguaje de programacion a SQL: excelente para el 90% de casos comunes (filtros, joins, ordenamiento), pero cuando necesitas expresar un modismo muy especifico del idioma nativo (Postgres), a veces es mas claro simplemente escribir esa frase directamente en el idioma original (SQL crudo) en vez de forzar una traduccion torpe.

## Ejemplo base

```typescript
@Injectable()
export class ProductosRepository extends Repository<Producto> {
  constructor(private dataSource: DataSource) {
    super(Producto, dataSource.createEntityManager());
  }

  // QueryBuilder para un caso relativamente comun
  async buscarPorCategoria(categoria: string) {
    return this.createQueryBuilder('p')
      .where('p.categoria = :categoria', { categoria })
      .orderBy('p.creadoEn', 'DESC')
      .getMany();
  }

  // SQL crudo para aprovechar un operador especifico de Postgres (JSONB)
  async buscarConMetadata(filtro: Record<string, any>) {
    return this.query(
      `SELECT * FROM productos WHERE metadata @> $1::jsonb`,
      [JSON.stringify(filtro)],
    );
  }

  // SQL crudo para Full Text Search (ver capitulo de Tipos Avanzados)
  async buscarPorTexto(termino: string) {
    return this.query(
      `SELECT *, ts_rank(busqueda, query) AS relevancia
       FROM productos, to_tsquery('spanish', $1) AS query
       WHERE busqueda @@ query
       ORDER BY relevancia DESC`,
      [termino],
    );
  }
}
```

## Como se usa en un proyecto real

1. Se usa el QueryBuilder para la mayoria de consultas complejas comunes (filtros dinamicos, joins, agregaciones), manteniendo tipado y legibilidad.
2. Se usa SQL crudo con `.query()` parametrizado (nunca concatenacion de strings, ver el mismo criterio de seguridad ya visto en varios libros) cuando se aprovecha una feature especifica de Postgres.
3. Se centraliza esta logica en repositorios custom, siguiendo el mismo principio de organizacion ya visto en el capitulo de Entidades del libro de NestJS.
4. Se testea el repositorio custom con casos reales, aplicando el mismo enfoque de testing de servicios ya visto en el libro de NestJS.
5. Se documenta cada metodo de SQL crudo explicando por que el QueryBuilder no era suficiente, ayudando a futuros mantenedores.

## Criterio junior

Un junior debe poder usar el QueryBuilder de TypeORM para una consulta con filtros y ordenamiento basico.

## Criterio mid

Un mid debe crear un repositorio custom con metodos que combinan QueryBuilder y SQL crudo parametrizado segun corresponda.

## Criterio senior

Un senior debe establecer el criterio del equipo sobre cuando usar QueryBuilder versus SQL crudo, y revisar en code review que el SQL crudo este siempre correctamente parametrizado sin riesgo de inyeccion.

## Errores comunes

- Forzar el QueryBuilder a expresar una feature muy especifica de Postgres de forma torpe cuando SQL crudo seria mas simple y legible.
- Concatenar valores directamente en SQL crudo en vez de parametrizarlos, arriesgando inyeccion SQL.
- No centralizar queries complejas en repositorios custom, dispersando SQL crudo por todo el codigo de servicios.
- No testear metodos de repositorio con SQL crudo, confiando en que "funcionan" sin verificacion sistematica.

## Practica

- [ ] Crea un repositorio custom con un metodo usando QueryBuilder para un filtro complejo.
- [ ] Agrega un metodo con SQL crudo parametrizado que aproveche una feature especifica de Postgres (JSONB o Full Text Search).
- [ ] Verifica que el SQL crudo esta correctamente parametrizado, sin concatenacion de valores de usuario.
- [ ] Escribe un test para el repositorio custom completo.
- [ ] Documenta el criterio de tu equipo sobre cuando usar QueryBuilder versus SQL crudo.

## Preguntas de repaso

- Cuando conviene usar SQL crudo en vez de forzar el QueryBuilder de TypeORM?
- Por que parametrizar valores en SQL crudo es indispensable para seguridad?
- Que ventaja de organizacion da centralizar queries complejas en un repositorio custom?
- Como testearias un metodo de repositorio que usa SQL crudo?
