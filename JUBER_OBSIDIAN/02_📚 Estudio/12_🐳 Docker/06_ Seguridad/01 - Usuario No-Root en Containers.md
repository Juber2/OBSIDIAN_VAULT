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

# 01 - Usuario No-Root en Containers

## Que vas a aprender

En este capitulo vas a aprender por que correr procesos como **root dentro de un container** es un riesgo de seguridad, y como configurar un usuario no-root correctamente en el Dockerfile.

## Explicacion

Por defecto, si no se especifica lo contrario, el proceso principal de un container corre como **root** dentro del namespace del container. Aunque el aislamiento de namespaces limita bastante el impacto, root dentro del container sigue teniendo mas capacidades que un usuario normal, y ciertas vulnerabilidades de escape de container (bugs en el kernel o mala configuracion) tienen mucho mayor impacto si el proceso comprometido corria como root.

La practica recomendada es crear un usuario y grupo especificos en el Dockerfile, asignar los permisos necesarios sobre los archivos de la aplicacion, y usar la instruccion `USER` para que el proceso principal corra con privilegios minimos, siguiendo el principio de menor privilegio.

## Modelo mental

Pensa correr como root dentro de un container como darle a un empleado nuevo las llaves maestras de todo el edificio en su primer dia, "por si acaso". Si ese empleado comete un error o es engañado (el equivalente a una vulnerabilidad explotada), el daño potencial es mucho mayor que si solo tuviera la llave de su propia oficina.

## Ejemplo base

```dockerfile
FROM node:20-alpine
RUN addgroup -g 1001 app && adduser -D -u 1001 -G app app
WORKDIR /app
COPY --chown=app:app package*.json ./
RUN npm ci --omit=dev
COPY --chown=app:app . .
USER app
CMD ["node", "dist/main.js"]
```

```bash
docker run --rm mi-imagen whoami
# app (no root)
```

## Como se usa en un proyecto real

1. Se crea un usuario no-root explicito en cada Dockerfile de aplicacion propia, con UID fijo y conocido (ver capitulo de Volumenes en produccion).
2. Se usa `--chown` en `COPY` para asegurar que los archivos copiados pertenezcan al usuario correcto desde el principio, evitando pasos adicionales de `chown`.
3. Se verifica, para imagenes base de terceros, si ya proveen un usuario no-root predefinido (muchas imagenes oficiales lo hacen) antes de crear uno propio innecesariamente.
4. Se documenta como excepcion justificada cualquier caso donde un proceso realmente necesite correr como root (poco comun, mayormente asociado a necesitar bindear puertos privilegiados menores a 1024).
5. Se verifica en CI/CD, como parte de un escaneo de seguridad, que las imagenes construidas no corren como root por defecto.

## Criterio junior

Un junior debe poder crear un usuario no-root en un Dockerfile y usar `USER` para que el proceso principal corra con el.

## Criterio mid

Un mid debe usar `--chown` correctamente al copiar archivos, y verificar si una imagen base de terceros ya provee un usuario no-root apropiado.

## Criterio senior

Un senior debe establecer como estandar obligatorio del equipo que ninguna imagen de produccion corra como root, integrando esta verificacion en el pipeline de CI/CD como gate automatizado.

## Errores comunes

- Dejar el usuario por defecto (root) sin ninguna justificacion, simplemente por no agregar el paso extra de crear un usuario.
- Usar `chown -R` como instruccion separada despues de `COPY` en vez de `--chown` directo, generando una capa adicional innecesaria.
- No verificar si la imagen base ya provee un usuario no-root utilizable antes de crear uno redundante.
- No automatizar la verificacion de "no corre como root" en CI, dependiendo solo de que el desarrollador se acuerde de revisarlo manualmente.

## Practica

- [ ] Crea un usuario no-root en un Dockerfile de practica y verifica con `whoami` dentro del container.
- [ ] Usa `--chown` en `COPY` para asignar permisos correctos sin un paso adicional.
- [ ] Investiga si alguna imagen base que uses ya provee un usuario no-root predefinido.
- [ ] Audita un Dockerfile existente de tu proyecto y corrige si corre como root sin justificacion.
- [ ] Anota como automatizarias la verificacion de "no corre como root" en tu pipeline de CI.

## Preguntas de repaso

- Por que correr un proceso como root dentro de un container es mas riesgoso que correrlo como un usuario normal, aun con el aislamiento de namespaces?
- Que ventaja da usar `--chown` en `COPY` sobre un `RUN chown` separado?
- Como verificarias si una imagen base de terceros ya provee un usuario no-root apropiado?
- Que caso justificaria, de forma excepcional, que un proceso corra como root dentro de un container?
