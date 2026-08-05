---
tags:
  - NestJS
  - TypeScript
  - Nest-Arquitectura
Creado: 2026-07-27
Relacionado:
  - Arquitectura NestJS
  - NESTJS
---

# 01 - Separacion de Responsabilidades por Capas

## Que vas a aprender

En este capitulo vas a consolidar el modelo de capas que ya veniste aplicando de forma implicita en capitulos anteriores: **controller (entrada), servicio (logica de aplicacion), repositorio (datos)**, y por que respetar esos limites es la base de cualquier arquitectura mas avanzada.

## Explicacion

La separacion en capas no es dogma por dogma: existe para que cada pieza tenga una sola razon para cambiar. Un controller cambia si cambia la forma de exponer algo por HTTP. Un servicio cambia si cambia una regla de negocio. Un repositorio cambia si cambia como se persisten los datos. Cuando estas responsabilidades se mezclan, cualquier cambio chico obliga a tocar y re-testear codigo que no tenia nada que ver.

Esta capa es la base sobre la que se construyen patrones mas sofisticados (DDD, CQRS, Clean Architecture) vistos mas adelante en este capitulo: sin disciplina en la separacion basica, esos patrones mas avanzados solo agregan complejidad sin beneficio real.

## Modelo mental

Pensa la aplicacion como una linea de ensamblaje: cada estacion (capa) hace una sola cosa bien definida y le pasa el resultado a la siguiente. Si una estacion empieza a hacer el trabajo de otras dos, la linea entera se vuelve dificil de mantener y de escalar con mas gente trabajando en paralelo.

## Ejemplo base

```typescript
// Controller: solo traduce HTTP <-> llamadas tipadas
@Post()
create(@Body() dto: CreateOrderDto) {
  return this.ordersService.create(dto);
}

// Servicio: logica de aplicacion y reglas de negocio
@Injectable()
export class OrdersService {
  constructor(private ordersRepo: OrdersRepository) {}

  async create(dto: CreateOrderDto) {
    if (dto.items.length === 0) throw new BadRequestException('Orden vacia');
    return this.ordersRepo.save(dto);
  }
}

// Repositorio: solo sabe hablar con la persistencia
@Injectable()
export class OrdersRepository {
  constructor(@InjectRepository(Order) private repo: Repository<Order>) {}
  save(data: Partial<Order>) { return this.repo.save(data); }
}
```

## Como se usa en un proyecto real

1. El controller nunca contiene reglas de negocio ni accede a repositorios directamente.
2. El servicio nunca conoce detalles de HTTP (status codes, headers) mas alla de las excepciones que lanza.
3. El repositorio nunca contiene reglas de negocio, solo operaciones de persistencia.
4. Se revisa en code review cualquier PR donde una capa empiece a "invadir" responsabilidades de otra.
5. Esta base se documenta como convencion minima del equipo, independiente de si despues se adopta DDD o Clean Architecture.

## Criterio junior

Un junior debe poder identificar, dado un archivo, si es un controller, servicio o repositorio, y notar cuando algo esta en el lugar equivocado.

## Criterio mid

Un mid debe refactorizar codigo existente que mezcla capas, moviendo cada responsabilidad a su lugar correcto sin romper comportamiento.

## Criterio senior

Un senior debe definir y hacer cumplir la convencion de capas del equipo, detectar tempranamente cuando un modulo esta creciendo de forma que necesita una capa adicional (por ejemplo, un "application service" separado de la logica de dominio pura).

## Errores comunes

- Controllers que acceden a repositorios directamente, saltandose el servicio.
- Servicios que arman respuestas HTTP (status codes, formato de error especifico de HTTP) en vez de solo lanzar excepciones de dominio.
- Repositorios con metodos que contienen reglas de negocio ("buscar usuarios activos que pueden comprar"), mezclando persistencia con logica.
- Dejar crecer un servicio indefinidamente sin dividirlo cuando acumula demasiadas responsabilidades distintas.

## Practica

- [ ] Revisa un proyecto existente y marca que archivos mezclan responsabilidades de mas de una capa.
- [ ] Refactoriza un controller que accede a un repositorio directo para que pase por un servicio.
- [ ] Extrae una regla de negocio de un repositorio hacia el servicio correspondiente.
- [ ] Define una checklist de code review para verificar separacion de capas.
- [ ] Anota un ejemplo real donde mezclar capas te costo tiempo de mantenimiento.

## Preguntas de repaso

- Que responsabilidad especifica tiene cada una de las tres capas basicas?
- Que señal te indica que un servicio esta creciendo demasiado y necesita dividirse?
- Por que un repositorio no deberia contener reglas de negocio?
- Como revisarias en code review si una capa esta invadiendo responsabilidades de otra?
