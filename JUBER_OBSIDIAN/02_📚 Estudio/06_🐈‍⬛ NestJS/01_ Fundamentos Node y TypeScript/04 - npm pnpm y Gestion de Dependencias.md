---
tags:
  - NestJS
  - TypeScript
  - Nest-Fundamentos
Creado: 2026-07-27
Relacionado:
  - Fundamentos Node y TypeScript
  - NESTJS
---

# 04 - npm pnpm y Gestion de Dependencias

## Que vas a aprender

En este capitulo vas a repasar como manejar dependencias en un proyecto NestJS: diferencias entre **npm, yarn y pnpm**, versionado semantico, lockfiles, y como evitar los problemas de dependencias mas comunes en proyectos backend de larga vida.

## Explicacion

NestJS es agnostico al package manager, pero la eleccion importa a escala: `npm` es el default y mas universal, `pnpm` ahorra espacio en disco y es mas rapido en monorepos gracias a su almacenamiento por hardlinks, y `yarn` (berry) agrega workspaces y resolucion mas estricta. El lockfile (`package-lock.json`, `pnpm-lock.yaml`) es lo que garantiza builds reproducibles: sin el, "funciona en mi maquina" es casi garantizado.

En proyectos Nest reales tambien importa distinguir `dependencies` (lo que corre en produccion, como `@nestjs/core`) de `devDependencies` (lo que solo se usa en build/test, como `@nestjs/testing`, `typescript`, `jest`).

## Modelo mental

El lockfile es una foto exacta del arbol de dependencias en un momento dado. `package.json` dice "que quiero", el lockfile dice "que instale exactamente, con que versiones exactas de todo, incluidas las dependencias transitivas".

## Ejemplo base

```json
{
  "dependencies": {
    "@nestjs/common": "^11.0.0",
    "@nestjs/core": "^11.0.0"
  },
  "devDependencies": {
    "@nestjs/cli": "^11.0.0",
    "@nestjs/testing": "^11.0.0",
    "typescript": "^5.4.0"
  }
}
```

## Como se usa en un proyecto real

1. Se elige un package manager para todo el equipo y se documenta (evitar mezclar npm y pnpm en el mismo repo).
2. El lockfile se commitea siempre al repositorio.
3. Se usa `npm ci` (o equivalente) en CI/CD, nunca `npm install`, para respetar el lockfile exacto.
4. Se auditan dependencias periodicamente (`npm audit`, Dependabot/Renovate) por vulnerabilidades.
5. Se revisa el tamaño del `node_modules` y dependencias no usadas en revisiones de mantenimiento.

## Criterio junior

Un junior debe saber instalar dependencias, diferenciar `dependencies` de `devDependencies`, y entender por que el lockfile se commitea.

## Criterio mid

Un mid debe poder diagnosticar conflictos de versiones, actualizar dependencias de forma segura, y elegir entre npm/pnpm/yarn segun el contexto del proyecto (monorepo, tamaño de equipo, velocidad de CI).

## Criterio senior

Un senior debe definir la politica de actualizacion de dependencias del equipo (Renovate/Dependabot, cadencia, quien aprueba majors), y decidir la estrategia de monorepo vs multi-repo cuando el proyecto crece.

## Errores comunes

- No commitear el lockfile y sufrir instalaciones inconsistentes entre desarrolladores.
- Usar `npm install` en CI en vez de `npm ci`, permitiendo drift silencioso de versiones.
- Mezclar package managers en el mismo repositorio.
- Dejar dependencias desactualizadas por años y terminar con una migracion mayor forzada y riesgosa.

## Practica

- [ ] Revisa el `package.json` de un proyecto Nest y separa mentalmente deps de produccion vs desarrollo.
- [ ] Corre una auditoria de seguridad de dependencias.
- [ ] Configura `npm ci` en un pipeline de CI de prueba.
- [ ] Investiga como configurar Renovate o Dependabot en un repo.
- [ ] Anota una politica propia de actualizacion de dependencias.

## Preguntas de repaso

- Que diferencia hay entre `npm install` y `npm ci`?
- Por que el lockfile debe commitearse?
- Que riesgo trae no auditar dependencias periodicamente?
- Cuando elegirias pnpm sobre npm?
