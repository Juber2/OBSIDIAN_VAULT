---
tags:
  - NestJS
  - TypeScript
  - Nest-Datos
Creado: 2026-07-27
Relacionado:
  - TypeORM Prisma y Base de Datos
  - NESTJS
---

# 02 - Entidades y Repositorios TypeORM

## Que vas a aprender

En este capitulo vas a aprender a modelar **entidades** con decoradores de TypeORM, y a usar el **patron Repository** que Nest integra via `TypeOrmModule.forFeature()` e inyeccion con `@InjectRepository()`.

## Explicacion

Una entidad TypeORM es una clase decorada con `@Entity()` que mapea una tabla; sus columnas se declaran con `@Column()`, `@PrimaryGeneratedColumn()`, etc. El **repositorio** es el objeto que da acceso a operaciones CRUD tipadas sobre esa entidad (`find`, `findOne`, `save`, `delete`), y se inyecta en un servicio con `@InjectRepository(Entidad)`.

La practica recomendada es que **solo el servicio** (o una capa de repositorio custom encima) hable con el repositorio de TypeORM; el controller nunca debe importar ni usar un repositorio directamente, porque eso rompe la separacion de capas.

## Modelo mental

Pensa la entidad como el plano de una tabla escrito en TypeScript, y el repositorio como el "empleado de mostrador" que sabe hablar con esa tabla: le pedis operaciones (buscar, guardar, borrar) y el traduce eso a SQL por vos.

## Ejemplo base

```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @CreateDateColumn()
  createdAt: Date;
}

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User) private readonly usersRepo: Repository<User>,
  ) {}

  findByEmail(email: string) {
    return this.usersRepo.findOne({ where: { email } });
  }
}
```

## Como se usa en un proyecto real

1. Cada entidad vive junto a su modulo de dominio, en una carpeta `entities/`.
2. Se registra con `TypeOrmModule.forFeature([User])` dentro del modulo correspondiente.
3. El acceso a datos siempre pasa por un servicio o repositorio custom, nunca directo desde el controller.
4. Se evita exponer la entidad de TypeORM directamente en la respuesta HTTP; se mapea a un DTO de salida.
5. Para queries complejas, se crea un repositorio custom que extiende el repositorio base en vez de amontonar logica en el servicio.

## Criterio junior

Un junior debe poder definir una entidad simple con columnas basicas, inyectar su repositorio, y hacer operaciones CRUD basicas.

## Criterio mid

Un mid debe diseñar entidades con tipos de columna correctos (indices, unicidad, defaults), y separar queries complejas en metodos de repositorio reutilizables en vez de repetir `find` con opciones enormes en cada servicio.

## Criterio senior

Un senior debe decidir cuando el patron Repository de TypeORM alcanza y cuando conviene una capa de abstraccion propia (Repository Pattern custom) para desacoplar el dominio del ORM, especialmente en arquitecturas hexagonales.

## Errores comunes

- Exponer la entidad de TypeORM tal cual en la respuesta HTTP, filtrando columnas sensibles (password, tokens) sin querer.
- Inyectar el repositorio directo en el controller, saltandose la capa de servicio.
- Repetir queries complejas casi identicas en varios servicios en vez de centralizarlas.
- No definir indices en columnas usadas frecuentemente en `WHERE`, degradando performance a medida que la tabla crece.

## Practica

- [ ] Define una entidad con al menos una columna unica y un indice.
- [ ] Crea un servicio que use el repositorio para operaciones CRUD.
- [ ] Mapea la entidad a un DTO de salida antes de devolverla por HTTP.
- [ ] Crea un metodo de repositorio custom para una query que se repite en varios lugares.
- [ ] Anota una regla propia: que nunca deberia devolver un endpoint directo desde una entidad.

## Preguntas de repaso

- Por que no conviene devolver una entidad de TypeORM directo en una respuesta HTTP?
- Que hace `@InjectRepository()` y donde deberia usarse?
- Cuando crearias un repositorio custom en vez de usar el repositorio base de TypeORM?
- Que problema de performance puede causar no indexar una columna muy consultada?
