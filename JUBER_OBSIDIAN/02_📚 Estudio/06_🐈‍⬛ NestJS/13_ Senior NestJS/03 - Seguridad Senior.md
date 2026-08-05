---
tags:
  - NestJS
  - TypeScript
  - Nest-Senior
Creado: 2026-07-27
Relacionado:
  - Senior NestJS
  - NESTJS
---

# 03 - Seguridad Senior

## Que vas a aprender

En este capitulo vas a consolidar una vision integral de seguridad para NestJS, mas alla de auth (ya visto en su propio capitulo): el **OWASP Top 10** aplicado a un backend Nest, y como pensar en seguridad como parte del diseño, no como una revision final.

## Explicacion

La seguridad senior no es una checklist que se corre al final; es una forma de pensar cada decision de diseño desde el principio. El OWASP Top 10 (inyeccion, autenticacion rota, exposicion de datos sensibles, control de acceso roto, mala configuracion de seguridad, entre otros) aplica directamente a un backend Nest: `class-validator` mitiga inyeccion en la capa de entrada, pero el ORM parametrizado (TypeORM/Prisma) es la defensa real contra SQL injection; los guards y CASL mitigan control de acceso roto; Helmet y CORS mitigan mala configuracion.

Un principio clave es **defensa en profundidad**: no confiar en una unica capa de proteccion. Por ejemplo, la autorizacion deberia verificarse tanto en el guard como, cuando aplica, en la query misma (filtrar por `userId` en el WHERE, no solo verificar el rol antes).

## Modelo mental

Pensa la seguridad como capas concentricas de un castillo, no como una unica puerta: si un atacante logra pasar una capa (por ejemplo, un bug en un guard), las capas siguientes (validacion a nivel de query, principio de menor privilegio en la base de datos) todavia lo detienen.

## Ejemplo base

```typescript
// Defensa en profundidad: autorizacion verificada en el guard Y en la query
@UseGuards(JwtAuthGuard)
@Get('mis-ordenes')
findMine(@CurrentUser() user: User) {
  // el guard verifica autenticacion, pero la query ademas filtra por dueño real
  return this.ordersRepo.find({ where: { userId: user.id } });
}

// Nunca construir queries con concatenacion de strings (SQL injection)
// Mal:
this.repo.query(`SELECT * FROM users WHERE email = '${email}'`);
// Bien: siempre parametrizado
this.repo.query('SELECT * FROM users WHERE email = $1', [email]);
```

## Como se usa en un proyecto real

1. Se aplica defensa en profundidad en operaciones sensibles: verificar autorizacion tanto en el guard como filtrando por dueño real en la query.
2. Nunca se construyen queries SQL con concatenacion de strings; siempre parametrizadas, incluso en queries raw ocasionales.
3. Se aplica principio de menor privilegio en credenciales de base de datos (el usuario de la app no deberia poder hacer `DROP TABLE`).
4. Se revisan dependencias periodicamente por vulnerabilidades conocidas (ver capitulo de gestion de dependencias).
5. Se realiza (o se contrata) una revision de seguridad antes de features criticas (pagos, datos personales sensibles) en vez de confiar solo en el review habitual.

## Criterio junior

Un junior debe conocer los riesgos basicos del OWASP Top 10 y evitar patrones obviamente inseguros (concatenar SQL, no validar entrada).

## Criterio mid

Un mid debe aplicar defensa en profundidad en operaciones sensibles y revisar dependencias por vulnerabilidades conocidas de forma proactiva.

## Criterio senior

Un senior debe pensar seguridad desde el diseño de cada feature (no como revision final), liderar auditorias de seguridad periodicas, y definir el proceso de respuesta ante un incidente de seguridad real.

## Errores comunes

- Confiar en una unica capa de proteccion (solo el guard, sin verificar dueño real en la query) para operaciones sensibles.
- Construir queries SQL con concatenacion de strings, incluso en queries raw ocasionales.
- Usar credenciales de base de datos con privilegios excesivos para la aplicacion.
- Tratar la seguridad como una revision de ultimo momento en vez de una consideracion de diseño desde el inicio de cada feature.

## Practica

- [ ] Revisa un endpoint sensible de tu proyecto y verifica que aplique defensa en profundidad.
- [ ] Audita el codigo en busca de queries SQL construidas con concatenacion de strings.
- [ ] Verifica los privilegios de la credencial de base de datos usada por la aplicacion.
- [ ] Corre una auditoria de dependencias por vulnerabilidades conocidas.
- [ ] Anota un proceso basico de respuesta ante un incidente de seguridad para tu equipo.

## Preguntas de repaso

- Que significa "defensa en profundidad" aplicado a autorizacion de datos?
- Por que concatenar strings para construir SQL es peligroso incluso con ORM disponible?
- Que es el principio de menor privilegio aplicado a credenciales de base de datos?
- Por que la seguridad deberia considerarse desde el diseño de una feature, no al final?
