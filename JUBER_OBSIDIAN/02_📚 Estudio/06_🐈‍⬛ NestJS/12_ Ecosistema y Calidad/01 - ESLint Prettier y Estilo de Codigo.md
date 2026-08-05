---
tags:
  - NestJS
  - TypeScript
  - Nest-Calidad
Creado: 2026-07-27
Relacionado:
  - Ecosistema y Calidad
  - NESTJS
---

# 01 - ESLint Prettier y Estilo de Codigo

## Que vas a aprender

En este capitulo vas a aprender a configurar **ESLint** y **Prettier** en un proyecto NestJS para mantener consistencia de estilo automatizada, y a integrarlos con Git hooks para que ningun codigo mal formateado llegue al repositorio.

## Explicacion

ESLint detecta problemas de codigo (variables no usadas, imports incorrectos, patrones peligrosos) mientras que Prettier se encarga exclusivamente del formato visual (indentacion, comillas, largo de linea), evitando discusiones de estilo en code review que no aportan valor real. El Nest CLI configura ambos por defecto al crear un proyecto, pero suele necesitar ajustes: reglas especificas para decoradores, integracion con `@typescript-eslint` para reglas conscientes de tipos.

La automatizacion completa incluye Git hooks (con `husky` + `lint-staged`) que corren lint y formato automaticamente en cada commit, para que ningun problema de estilo llegue siquiera a un Pull Request.

## Modelo mental

Pensa ESLint como un revisor que busca errores logicos y patrones riesgosos en el codigo, y Prettier como quien se encarga de que todo el codigo "se vea igual" sin que nadie tenga que discutirlo manualmente. Juntos liberan al equipo de discutir estilo en cada review, dejando esa energia para revisar logica real.

## Ejemplo base

```json
// .eslintrc.json (fragmento)
{
  "extends": ["plugin:@typescript-eslint/recommended", "plugin:prettier/recommended"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn"
  }
}
```

```json
// package.json con husky + lint-staged
{
  "lint-staged": {
    "*.ts": ["eslint --fix", "prettier --write"]
  }
}
```

## Como se usa en un proyecto real

1. Se configura ESLint y Prettier desde el inicio del proyecto, no como tarea "para despues".
2. Se integran con `husky` y `lint-staged` para que corran automaticamente antes de cada commit.
3. Se corre lint tambien en CI, como red de seguridad ante quien haga commit sin hooks activos (por ejemplo, un colaborador nuevo).
4. Se evita deshabilitar reglas de ESLint "para que compile" sin entender por que la regla existia.
5. Se documentan las reglas custom del equipo (si las hay) y su justificacion, para que no parezcan arbitrarias.

## Criterio junior

Un junior debe poder correr lint y formato localmente, y entender la diferencia de proposito entre ESLint y Prettier.

## Criterio mid

Un mid debe configurar Git hooks para automatizar lint y formato en cada commit, y ajustar reglas de ESLint segun las necesidades reales del proyecto.

## Criterio senior

Un senior debe definir el set de reglas de ESLint del equipo con criterio (no copiar configuraciones genericas sin evaluarlas), integrar lint en CI como gate obligatorio, y liderar la decision de cuando una regla vale la pena versus cuando genera friccion sin beneficio.

## Errores comunes

- Deshabilitar reglas de ESLint sin entender el problema que prevenian, dejando pasar bugs que la regla hubiera detectado.
- No integrar lint en CI, confiando solo en que cada desarrollador lo corra localmente.
- Discutir estilo de codigo en code review en vez de dejar que Prettier lo resuelva automaticamente.
- Copiar una configuracion de ESLint genérica de internet sin adaptarla a las necesidades reales del proyecto.

## Practica

- [ ] Configura ESLint y Prettier en un proyecto Nest, ajustando al menos una regla segun tu criterio.
- [ ] Integra husky y lint-staged para correr lint y formato en cada commit.
- [ ] Agrega el paso de lint al pipeline de CI.
- [ ] Revisa una regla deshabilitada en un proyecto existente y evalua si deberia reactivarse.
- [ ] Documenta las reglas custom de tu equipo y su justificacion.

## Preguntas de repaso

- Que responsabilidad tiene ESLint y que responsabilidad tiene Prettier?
- Por que integrar lint en CI, ademas de en Git hooks locales?
- Que riesgo tiene deshabilitar una regla de ESLint sin entender por que existia?
- Como decidirias si vale la pena agregar una regla custom de ESLint para tu equipo?
