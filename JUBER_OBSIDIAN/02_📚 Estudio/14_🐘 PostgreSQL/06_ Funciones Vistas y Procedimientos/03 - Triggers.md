---
tags:
  - PostgreSQL
  - Funciones
  - Postgres-Funciones
Creado: 2026-07-29
Relacionado:
  - Funciones Vistas y Procedimientos
  - POSTGRESQL
---

# 03 - Triggers

## Que vas a aprender

En este capitulo vas a aprender a crear **triggers**: funciones que se ejecutan automaticamente ante eventos de modificacion de datos (`INSERT`, `UPDATE`, `DELETE`), el caso de uso mas comun (actualizar `actualizado_en` automaticamente, ya mencionado en el capitulo de Modelado), y cuando triggers son la herramienta correcta versus cuando no.

## Explicacion

Un trigger asocia una funcion (escrita tipicamente en PL/pgSQL) a un evento especifico sobre una tabla: `BEFORE INSERT` (se ejecuta antes de insertar, puede modificar los valores a insertar), `AFTER UPDATE` (se ejecuta despues de actualizar, util para efectos secundarios como auditoria), `BEFORE DELETE`, etc. Dentro de la funcion del trigger, `NEW` referencia la fila que se esta insertando/actualizando, y `OLD` referencia la fila anterior (relevante en `UPDATE`/`DELETE`).

El caso de uso mas solido para triggers es mantener **invariantes automaticas** que deben cumplirse sin importar por donde entren los datos: actualizar `actualizado_en` en cada `UPDATE`, mantener una tabla de auditoria de cambios, o validar una regla compleja que no se puede expresar como `CHECK` simple. Triggers deben usarse con moderacion: logica de negocio compleja escondida en triggers es dificil de descubrir, testear y depurar, siendo preferible mantenerla explicita en la capa de aplicacion cuando no hay una razon real de "invariante automatica" que lo justifique.

## Modelo mental

Pensa un trigger como una alarma silenciosa automatica que se dispara sola cada vez que algo especifico pasa (alguien entra por esa puerta), sin que nadie tenga que acordarse de activarla manualmente cada vez. Es perfecto para reglas que **siempre** deben cumplirse sin excepcion, pero si empezas a poner demasiada logica compleja en alarmas silenciosas escondidas, se vuelve dificil para cualquiera entender todo lo que realmente pasa cuando alguien simplemente abre una puerta.

## Ejemplo base

```sql
CREATE FUNCTION actualizar_timestamp() RETURNS trigger AS $$
BEGIN
    NEW.actualizado_en = now();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_pedidos_actualizado
BEFORE UPDATE ON pedidos
FOR EACH ROW
EXECUTE FUNCTION actualizar_timestamp();

-- Trigger de auditoria
CREATE TABLE auditoria_pedidos (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    pedido_id bigint,
    accion text,
    datos_anteriores jsonb,
    ejecutado_en timestamptz DEFAULT now()
);

CREATE FUNCTION registrar_auditoria() RETURNS trigger AS $$
BEGIN
    INSERT INTO auditoria_pedidos (pedido_id, accion, datos_anteriores)
    VALUES (OLD.id, TG_OP, to_jsonb(OLD));
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_auditoria_pedidos
AFTER UPDATE OR DELETE ON pedidos
FOR EACH ROW
EXECUTE FUNCTION registrar_auditoria();
```

## Como se usa en un proyecto real

1. Se usa un trigger `BEFORE UPDATE` estandar para actualizar `actualizado_en` automaticamente en cualquier tabla, aplicado como patron reutilizable en todo el esquema.
2. Se usan triggers `AFTER` para efectos secundarios de auditoria que deben capturar cambios sin importar el camino de acceso (API, script directo, migracion).
3. Se evita poner logica de negocio compleja de la aplicacion en triggers, prefiriendo mantenerla explicita y testeable en NestJS salvo una razon real de invariante automatica.
4. Se documenta claramente cada trigger existente en un esquema, dado que su comportamiento no es visible simplemente leyendo el codigo de la aplicacion.
5. Se testean triggers con casos SQL directos, verificando que el efecto esperado ocurre ante cada tipo de evento que disparan.

## Criterio junior

Un junior debe poder crear un trigger simple que actualiza una columna automaticamente ante un evento.

## Criterio mid

Un mid debe crear triggers de auditoria usando `NEW`/`OLD` y `TG_OP`, y documentar su existencia claramente para el equipo.

## Criterio senior

Un senior debe decidir con criterio cuando un trigger es la herramienta correcta (invariantes automaticas) versus cuando la logica deberia vivir explicitamente en la aplicacion, evitando "magia oculta" excesiva en el esquema.

## Errores comunes

- Esconder logica de negocio compleja en triggers, haciendola dificil de descubrir y depurar para quien no sabe que el trigger existe.
- No documentar triggers existentes, generando comportamiento "misterioso" que sorprende a nuevos integrantes del equipo.
- Usar triggers para casos donde un simple `CHECK` constraint (ver capitulo de Modelado) ya resolveria la necesidad de forma mas simple.
- No testear triggers con casos SQL directos, confiando en que "funcionan" sin verificacion sistematica.

## Practica

- [ ] Crea un trigger `BEFORE UPDATE` que actualice `actualizado_en` automaticamente.
- [ ] Crea un trigger de auditoria que registre cambios en una tabla de auditoria separada.
- [ ] Prueba el trigger ejecutando un `UPDATE` y verificando el efecto esperado.
- [ ] Documenta todos los triggers de un esquema de practica en un lugar centralizado.
- [ ] Evalua un caso donde estarias tentado a usar un trigger, y decide si en realidad seria mejor logica explicita en NestJS.

## Preguntas de repaso

- Que diferencia hay entre `NEW` y `OLD` dentro de una funcion de trigger?
- Que caso de uso es el mas solido para justificar un trigger (invariante automatica)?
- Por que la logica de negocio compleja en triggers puede ser problematica a largo plazo?
- Como documentarias la existencia de triggers en un esquema para que no sean "magia oculta" para el equipo?
