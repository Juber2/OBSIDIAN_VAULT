---
tags:
  - Docker
  - DevOps
  - Docker-Seguridad
Creado: 2026-07-28
Relacionado:
  - Seguridad
  - DOCKER
---

# 05 - Imagenes de Fuentes Confiables

## Que vas a aprender

En este capitulo vas a aprender a evaluar la **confiabilidad de una imagen base** antes de usarla: imagenes oficiales vs comunidad vs desconocidas, y practicas para reducir el riesgo de una cadena de suministro de software comprometida.

## Explicacion

No todas las imagenes en Docker Hub tienen el mismo nivel de confianza. Las **imagenes oficiales** (mantenidas por el proyecto o por Docker directamente, como `node`, `postgres`, `nginx`) pasan por procesos de revision y actualizacion regulares. Las imagenes de **usuarios o organizaciones verificadas** varian en calidad segun quien las mantiene. Usar una imagen de un usuario desconocido, sin verificar, es un vector de riesgo real: una imagen maliciosa podria contener malware, backdoors, o mineros de criptomonedas ocultos.

Este es un caso concreto de riesgo de **cadena de suministro de software** (supply chain security): confias no solo en tu propio codigo, sino en cada capa de cada imagen base que usas, y en cada dependencia que esas imagenes instalan. Fijar versiones exactas (o mejor, digests especificos por hash) reduce el riesgo de que una imagen "confiable" cambie de contenido sin que te des cuenta.

## Modelo mental

Pensa usar una imagen base como comprar un ingrediente para tu receta: comprarlo en un proveedor reconocido y con controles de calidad (imagen oficial) es mucho mas seguro que comprarlo suelto, sin marca, en un puesto callejero desconocido (imagen sin verificar de un usuario cualquiera), aunque ambos puedan verse identicos a simple vista.

## Ejemplo base

```dockerfile
# Preferido: imagen oficial, version fijada
FROM node:20.11.1-alpine3.19

# Aun mas estricto: fijado por digest exacto (inmutable, no puede cambiar el contenido)
FROM node@sha256:abcdef1234567890...
```

```bash
# Verificar el digest actual de una imagen antes de fijarlo en el Dockerfile
docker inspect --format '{{index .RepoDigests 0}}' node:20-alpine
```

## Como se usa en un proyecto real

1. Se prioriza siempre imagenes oficiales o de organizaciones verificadas para componentes de infraestructura (bases de datos, lenguajes, servidores web).
2. Se fija la version exacta de cada imagen base usada, y para casos de alta sensibilidad, se fija por digest en vez de por tag (los tags pueden reapuntar a contenido distinto, los digests no).
3. Se evita usar imagenes de usuarios desconocidos sin revisar su Dockerfile fuente (si esta disponible publicamente) antes de confiar en ellas.
4. Se combina esto con el escaneo de vulnerabilidades (ver tema anterior) como capas complementarias: una imagen de fuente confiable tambien puede tener vulnerabilidades conocidas que hay que parchear.
5. Se documenta la fuente y justificacion de cada imagen base no trivial usada en el proyecto.

## Criterio junior

Un junior debe poder distinguir una imagen oficial de una imagen de usuario no verificado en Docker Hub antes de usarla.

## Criterio mid

Un mid debe fijar versiones exactas de imagenes base y evaluar criticamente la fuente antes de introducir una imagen nueva de terceros al proyecto.

## Criterio senior

Un senior debe definir la politica de fuentes de imagenes aprobadas para el equipo, considerando fijado por digest para componentes criticos, y evaluar riesgos de cadena de suministro como parte de la revision de seguridad del sistema.

## Errores comunes

- Usar una imagen de un usuario desconocido sin ninguna verificacion solo porque resuelve un problema puntual rapido.
- Fijar solo por tag (`node:20`) sin considerar que un tag puede reapuntar a contenido distinto con el tiempo, a diferencia de un digest inmutable.
- No revisar el Dockerfile fuente de una imagen de terceros cuando esta disponible, perdiendo la oportunidad de auditar que realmente contiene.
- Confiar ciegamente en que "imagen oficial" significa "sin vulnerabilidades", sin complementar con escaneo activo.

## Practica

- [ ] Identifica, en Docker Hub, la diferencia visual entre una imagen oficial y una de un usuario cualquiera.
- [ ] Fija una imagen base de tu proyecto por digest exacto en vez de solo por tag.
- [ ] Revisa el Dockerfile fuente (si esta disponible) de una imagen de terceros que uses.
- [ ] Combina el fijado por digest con un escaneo de vulnerabilidades sobre esa misma imagen.
- [ ] Documenta la politica de fuentes de imagenes aprobadas para tu equipo.

## Preguntas de repaso

- Que diferencia hay entre una imagen oficial y una imagen de un usuario no verificado en Docker Hub?
- Por que fijar por digest es mas seguro que fijar solo por tag?
- Que riesgo de cadena de suministro implica usar una imagen de una fuente no confiable?
- Por que el fijado de version por si solo no reemplaza al escaneo de vulnerabilidades?
