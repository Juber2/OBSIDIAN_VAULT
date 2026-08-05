---
tags:
  - NestJS
  - TypeScript
  - Nest-Auth
Creado: 2026-07-27
Relacionado:
  - Auth y Seguridad NestJS
  - NESTJS
---

# 01 - Autenticacion con Passport

## Que vas a aprender

En este capitulo vas a aprender a integrar **Passport** con NestJS (`@nestjs/passport`) para implementar autenticacion, entendiendo el patron de **strategies** (local, JWT) y como Nest los conecta con guards.

## Explicacion

Passport es la libreria de autenticacion mas usada en el ecosistema Node, basada en el concepto de **strategy**: cada estrategia (local con usuario/password, JWT, OAuth de Google/GitHub) implementa una forma especifica de validar credenciales. NestJS envuelve cada strategy en una clase que extiende `PassportStrategy`, y la conecta al request con un `AuthGuard('nombre-strategy')`.

El flujo tipico es: `LocalStrategy` valida usuario/password en el login y devuelve el usuario; el servicio de auth genera un JWT con ese usuario; a partir de ahi, `JwtStrategy` valida el token en cada request protegido y adjunta el usuario decodificado a `request.user`.

## Modelo mental

Pensa cada strategy de Passport como un "verificador de identidad" especializado: uno sabe verificar usuario y contraseña, otro sabe verificar un token JWT, otro sabe hablar con Google. Vos eliges cual usar en cada punto de entrada con `@UseGuards(AuthGuard('jwt'))`.

## Ejemplo base

```typescript
@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super({ usernameField: 'email' });
  }

  async validate(email: string, password: string) {
    const user = await this.authService.validateUser(email, password);
    if (!user) throw new UnauthorizedException();
    return user;
  }
}

@UseGuards(AuthGuard('local'))
@Post('login')
login(@Request() req) {
  return this.authService.login(req.user);
}
```

## Como se usa en un proyecto real

1. `LocalStrategy` se usa solo en el endpoint de login, para verificar credenciales una unica vez.
2. `JwtStrategy` protege todos los endpoints que requieren usuario autenticado, validando el token en cada request.
3. El `validate()` de cada strategy nunca expone la contraseña ni datos sensibles en lo que devuelve.
4. Se separan claramente los guards de autenticacion (¿quien sos?) de los guards de autorizacion (¿que podes hacer?).
5. Se testean las strategies de forma aislada, mockeando el servicio de autenticacion.

## Criterio junior

Un junior debe poder implementar un login basico con `LocalStrategy` y proteger un endpoint con `AuthGuard('jwt')`.

## Criterio mid

Un mid debe implementar ambas strategies (local y JWT) coordinadas en un flujo de login completo, y manejar correctamente errores de autenticacion sin filtrar informacion (por ejemplo, no revelar si el error fue "usuario no existe" vs "contraseña incorrecta").

## Criterio senior

Un senior debe diseñar la arquitectura completa de autenticacion del sistema (que strategies existen, como interactuan con auth externo tipo OAuth, como se maneja el logout con JWT stateless) y definir estandares de seguridad para el equipo.

## Errores comunes

- Devolver mensajes de error distintos para "usuario no existe" y "contraseña incorrecta", facilitando enumeracion de usuarios validos.
- Incluir datos sensibles (password hasheado, tokens internos) en lo que `validate()` adjunta a `request.user`.
- No separar claramente autenticacion de autorizacion, mezclando ambas responsabilidades en un solo guard.
- No testear las strategies de forma aislada, descubriendo bugs de auth solo en integracion.

## Practica

- [ ] Implementa `LocalStrategy` para un login con email y password.
- [ ] Implementa `JwtStrategy` y protege un endpoint con `AuthGuard('jwt')`.
- [ ] Verifica que los mensajes de error de login no revelen si el email existe o no.
- [ ] Escribe un test unitario para cada strategy, mockeando el servicio de auth.
- [ ] Anota que otras strategies (Google, GitHub) podrias necesitar en un proyecto real.

## Preguntas de repaso

- Que responsabilidad tiene una Passport strategy?
- Por que conviene no revelar si el error de login es de usuario o de contraseña?
- Que diferencia hay entre un guard de autenticacion y uno de autorizacion?
- Que datos nunca deberian terminar en `request.user`?
