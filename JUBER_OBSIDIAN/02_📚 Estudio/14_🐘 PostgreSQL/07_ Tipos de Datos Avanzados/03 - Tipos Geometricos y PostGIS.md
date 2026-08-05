---
tags:
  - PostgreSQL
  - Tipos-de-Datos
  - Postgres-Avanzado
Creado: 2026-07-29
Relacionado:
  - Tipos de Datos Avanzados
  - POSTGRESQL
---

# 03 - Tipos Geometricos y PostGIS

## Que vas a aprender

En este capitulo vas a conocer los tipos geometricos nativos de Postgres (`point`, `line`, `polygon`) y una introduccion conceptual a **PostGIS**, la extension que convierte a Postgres en una base de datos geoespacial de nivel profesional, ampliamente usada para funcionalidades de ubicacion.

## Explicacion

Postgres incluye tipos geometricos basicos de forma nativa (`point` para coordenadas, `polygon` para areas, `line`/`lseg` para lineas), suficientes para casos simples con operadores geometricos basicos (distancia, contencion). Para trabajo geoespacial real (calcular distancias considerando la curvatura de la Tierra, buscar "puntos cercanos" eficientemente, trabajar con sistemas de coordenadas geograficas estandar), se usa la extension **PostGIS** (ya mencionada en el capitulo de Extensiones), que agrega tipos como `geometry`/`geography`, funciones especializadas (`ST_Distance`, `ST_Within`, `ST_Intersects`), e indices GiST optimizados para consultas espaciales.

Este tipo de funcionalidad es relevante para aplicaciones con features de ubicacion: "tiendas cercanas a mi", "entregas dentro de una zona de cobertura", "puntos de interes dentro de un radio especifico", todas consultas que serian extremadamente ineficientes de calcular manualmente sin indices espaciales apropiados.

## Modelo mental

Pensa los tipos geometricos basicos de Postgres como una regla y compas simples: suficientes para dibujar y medir formas basicas. PostGIS es como un sistema de cartografia profesional completo: entiende que la Tierra es una esfera (no un plano), calcula distancias reales considerando eso, y tiene herramientas especializadas para preguntas complejas de ubicacion que un compas simple nunca podria resolver eficientemente.

## Ejemplo base

```sql
-- Tipos geometricos nativos, sin PostGIS
CREATE TABLE ubicaciones_simples (
    id serial PRIMARY KEY,
    coordenada point
);
SELECT * FROM ubicaciones_simples WHERE coordenada <-> point(0,0) < 10;  -- distancia euclidiana simple

-- Con PostGIS instalado
CREATE EXTENSION IF NOT EXISTS postgis;

CREATE TABLE tiendas (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre text,
    ubicacion geography(Point, 4326)   -- SRID 4326: coordenadas GPS estandar
);

INSERT INTO tiendas (nombre, ubicacion) VALUES
    ('Tienda Centro', ST_MakePoint(-58.3816, -34.6037));   -- longitud, latitud

-- Tiendas dentro de 5km de una ubicacion dada, con indice espacial
SELECT nombre FROM tiendas
WHERE ST_DWithin(ubicacion, ST_MakePoint(-58.38, -34.60)::geography, 5000);

CREATE INDEX idx_tiendas_ubicacion ON tiendas USING gist (ubicacion);
```

## Como se usa en un proyecto real

1. Se usan tipos geometricos nativos simples solo para casos muy basicos que no requieren precision geografica real (coordenadas en un plano simple, no en el globo terraqueo).
2. Se instala PostGIS para cualquier funcionalidad real de ubicacion (busqueda de cercania, zonas de cobertura, calculo de rutas basico) que necesite considerar la geografia real de la Tierra.
3. Se usa `geography` (no `geometry`) para datos GPS reales, dado que calcula distancias correctamente sobre la superficie curva de la Tierra en vez de asumir un plano.
4. Se indexa con GiST cualquier columna geoespacial consultada frecuentemente por cercania o contencion, indispensable para performance aceptable en tablas grandes.
5. Se verifica la disponibilidad de PostGIS antes de comprometerse con un proveedor cloud especifico (ver capitulo de Postgres en la Nube), dado que no todos los servicios gestionados la soportan.

## Criterio junior

Un junior debe conocer que existen tipos geometricos nativos y que PostGIS es la extension estandar para trabajo geoespacial serio.

## Criterio mid

Un mid debe usar PostGIS basico (`ST_MakePoint`, `ST_DWithin`) para funcionalidades de cercania, indexando apropiadamente con GiST.

## Criterio senior

Un senior debe decidir cuando una funcionalidad de ubicacion justifica la complejidad de PostGIS, y verificar su disponibilidad como requisito antes de elegir infraestructura cloud.

## Errores comunes

- Usar tipos geometricos nativos simples para calculos de distancia real sobre coordenadas GPS, obteniendo resultados incorrectos por no considerar la curvatura de la Tierra.
- Usar `geometry` en vez de `geography` para datos GPS reales, generando calculos de distancia imprecisos.
- No indexar con GiST una columna geoespacial consultada frecuentemente, generando consultas de cercania extremadamente lentas en tablas grandes.
- No verificar la disponibilidad de PostGIS en un proveedor cloud antes de comprometerse con esa plataforma para un proyecto que la necesita.

## Practica

- [ ] Instala PostGIS (si tu instalacion lo permite) y crea una tabla con una columna `geography`.
- [ ] Inserta coordenadas de ubicaciones reales y consulta cercania con `ST_DWithin`.
- [ ] Indexa la columna geoespacial con GiST y verifica su uso con `EXPLAIN ANALYZE`.
- [ ] Compara conceptualmente el resultado de un calculo de distancia con tipos nativos simples versus PostGIS.
- [ ] Investiga si tu proveedor cloud de preferencia soporta PostGIS.

## Preguntas de repaso

- Por que los tipos geometricos nativos simples de Postgres no son suficientes para calculos de distancia GPS reales?
- Que diferencia hay entre `geometry` y `geography` en PostGIS?
- Que tipo de indice se usa para columnas geoespaciales, y por que es indispensable para performance?
- Que verificarias antes de elegir un proveedor cloud para un proyecto que necesita PostGIS?
