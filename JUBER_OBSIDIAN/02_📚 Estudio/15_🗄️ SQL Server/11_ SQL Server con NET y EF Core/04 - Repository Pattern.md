---
tags:
  - SQLServer
  - DotNet
  - SQLServer-DotNet
Creado: 2026-07-29
Relacionado:
  - SQL Server con NET y EF Core
  - SQL SERVER
---

# 04 - Repository Pattern

## Que vas a aprender

En este capitulo vas a aprender el **Repository Pattern** aplicado sobre EF Core: una capa de abstraccion entre la logica de negocio y el acceso a datos, y el debate real sobre si sigue siendo necesario dado que `DbContext` ya implementa el patron Unit of Work + Repository internamente.

## Explicacion

El Repository Pattern envuelve el acceso a datos detras de una interfaz (`ICursoRepository` con metodos como `GetById`, `GetAll`, `Add`), ocultando los detalles de EF Core (o cualquier ORM) del resto de la aplicacion. El argumento clasico a favor es facilitar testing (mockear `ICursoRepository` en vez de `DbContext`) y desacoplar la logica de negocio del ORM especifico usado. El argumento en contra, sostenido por gran parte de la comunidad .NET moderna, es que `DbContext` **ya es** una implementacion de Unit of Work + Repository (cada `DbSet<T>` es efectivamente un repositorio), y agregar una capa adicional de Repository sobre EF Core a menudo solo añade indireccion sin beneficio real, especialmente en proyectos donde el ORM no va a cambiarse.

Un termino medio pragmatico, comun en proyectos reales como StaffTrainingPlatform (que ya sigue Clean Architecture con CQRS segun el proyecto), es usar directamente `DbContext` dentro de Handlers/Actions especificos de la capa de Infrastructure, exponiendo solo interfaces a nivel de caso de uso (no de entidad generica), evitando tanto el acoplamiento directo del dominio a EF Core como la indireccion excesiva de un Repository generico por entidad.

## Modelo mental

Pensa el debate del Repository Pattern como decidir si necesitas un traductor personal (el Repository) entre vos y alguien que ya habla tu idioma razonablemente bien (`DbContext`, que ya es un Repository/Unit of Work). Si nunca vas a cambiar de idioma (de ORM), el traductor adicional a veces solo agrega una capa de conversacion sin necesidad real; pero si necesitas simular conversaciones para practicar (testing) sin depender del interlocutor real, tener esa interfaz intermedia ayuda.

## Ejemplo base

```csharp
// Interfaz a nivel de caso de uso, no generica por entidad
public interface ICourseCrudActions
{
    Task<Curso> CrearCursoAsync(CrearCursoRequest request);
    Task<Curso?> ObtenerCursoAsync(int id);
}

// Implementacion en la capa de Infrastructure, usando DbContext directamente
public class CourseCrudActions : ICourseCrudActions
{
    private readonly AppDbContext _context;

    public CourseCrudActions(AppDbContext context) => _context = context;

    public async Task<Curso> CrearCursoAsync(CrearCursoRequest request)
    {
        var curso = new Curso { Nombre = request.Nombre };
        _context.Cursos.Add(curso);
        await _context.SaveChangesAsync();
        return curso;
    }
}
```

## Como se usa en un proyecto real

1. Se evalua en StaffTrainingPlatform si un Repository generico por entidad aporta valor real, o si exponer interfaces a nivel de caso de uso (como ya hace el patron CQRS con Actions) es suficiente sin la capa adicional.
2. Se usa `DbContext` directamente dentro de la capa de Infrastructure, respetando Clean Architecture al no exponer EF Core directamente hacia la capa de dominio o aplicacion.
3. Se testea la logica de negocio mockeando las interfaces de caso de uso (`ICourseCrudActions`), no un Repository generico innecesario.
4. Se evita el anti-patron de "Repository generico que solo envuelve `DbSet<T>` sin logica adicional", que la comunidad .NET moderna considera indireccion sin beneficio.
5. Se documenta la decision arquitectonica tomada (con o sin Repository Pattern) para que el equipo entienda el razonamiento, no solo copie el patron por convencion.

## Criterio junior

Un junior debe entender que es el Repository Pattern y por que existe el debate sobre su necesidad con EF Core.

## Criterio mid

Un mid debe poder implementar acceso a datos usando `DbContext` directamente dentro de una arquitectura por capas sin necesitar un Repository generico.

## Criterio senior

Un senior debe decidir con criterio arquitectonico si el proyecto se beneficia de Repository Pattern explicito o si `DbContext` dentro de la capa de Infrastructure es suficiente, documentando el porque.

## Errores comunes

- Implementar un Repository generico que solo envuelve `DbSet<T>` sin logica adicional, agregando indireccion sin beneficio real.
- Exponer `DbContext` directamente hacia la capa de dominio o aplicacion, violando la separacion de Clean Architecture.
- Copiar el patron por costumbre sin evaluar si realmente aporta valor en el contexto especifico del proyecto.
- No testear la logica de negocio adecuadamente, sea cual sea el patron de acceso a datos elegido.

## Practica

- [ ] Implementa acceso a datos usando `DbContext` directamente dentro de una capa de Infrastructure.
- [ ] Implementa el mismo caso de uso con un Repository generico, y compara la cantidad de codigo e indireccion.
- [ ] Analiza el codigo de StaffTrainingPlatform (Actions/Courses) e identifica que patron de acceso a datos usa actualmente.
- [ ] Diseña interfaces a nivel de caso de uso en vez de Repository generico por entidad.
- [ ] Documenta tu propio criterio de cuando usarias Repository Pattern explicito.

## Preguntas de repaso

- Por que `DbContext` ya es considerado una implementacion de Unit of Work + Repository?
- Que riesgo tiene un Repository generico que solo envuelve `DbSet<T>` sin logica adicional?
- Como se relaciona esta decision con Clean Architecture y CQRS ya usados en StaffTrainingPlatform?
- En que escenario justificarias agregar un Repository Pattern explicito sobre EF Core?
