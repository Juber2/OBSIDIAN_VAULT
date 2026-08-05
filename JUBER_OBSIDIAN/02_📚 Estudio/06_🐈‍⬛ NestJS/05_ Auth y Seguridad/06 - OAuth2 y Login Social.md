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

# 06 - OAuth2 y Login Social

## Que vas a aprender

En este capitulo vas a aprender a integrar **login social** (Google, GitHub, etc.) en NestJS usando strategies de Passport para OAuth2, y a entender el flujo de **Authorization Code** que hace posible ese login sin que tu app maneje contraseñas ajenas.

## Explicacion

OAuth2 es un protocolo de autorizacion (no autenticacion pura) que permite a un usuario darle permiso a tu aplicacion para acceder a cierta informacion de otro proveedor (Google, GitHub) sin compartir su contraseña con vos. El flujo mas comun para aplicaciones web es **Authorization Code**: el usuario es redirigido al proveedor, autoriza el acceso, el proveedor redirige de vuelta con un codigo, y tu backend intercambia ese codigo por un token de acceso.

En NestJS esto se implementa con strategies especificas de Passport (`passport-google-oauth20`, `passport-github2`), que manejan todo el intercambio de codigo por token, y en el `validate()` de la strategy vos decidis que hacer con el perfil recibido: crear un usuario nuevo si no existe, o vincularlo a uno existente.

## Modelo mental

Pensa OAuth2 como un valet parking: el usuario no te da las llaves de su auto (su contraseña de Google), le da las llaves a un tercero de confianza (Google) que te entrega un ticket (token) que solo sirve para lo que el usuario autorizo, nada mas.

## Ejemplo base

```typescript
@Injectable()
export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
  constructor() {
    super({
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: '/auth/google/callback',
      scope: ['email', 'profile'],
    });
  }

  async validate(accessToken: string, refreshToken: string, profile: Profile) {
    return { email: profile.emails[0].value, name: profile.displayName };
  }
}

@Get('google')
@UseGuards(AuthGuard('google'))
googleAuth() {}

@Get('google/callback')
@UseGuards(AuthGuard('google'))
googleCallback(@Request() req) {
  return this.authService.loginOrRegister(req.user);
}
```

## Como se usa en un proyecto real

1. Se registra la aplicacion en cada proveedor (Google Cloud Console, GitHub Developer Settings) y se obtienen `clientID`/`clientSecret`.
2. El callback del proveedor crea el usuario si no existe, o lo vincula a una cuenta existente por email.
3. Despues del login social exitoso, se emite el mismo access/refresh token JWT que el login tradicional, unificando el resto del sistema.
4. Se maneja el caso de un email que ya existe con login tradicional: se decide si se vincula automaticamente o se pide confirmacion.
5. Se testea el flujo completo en un ambiente de staging con credenciales de prueba del proveedor.

## Criterio junior

Un junior debe poder configurar una strategy de OAuth2 basica siguiendo la documentacion del proveedor y entender el flujo general de redireccion.

## Criterio mid

Un mid debe manejar correctamente el caso de vinculacion de cuentas (mismo email, distinto metodo de login) y unificar el token emitido despues de login social con el resto del sistema de auth.

## Criterio senior

Un senior debe diseñar la estrategia de identidad del sistema cuando conviven multiples proveedores de login, definir la politica de vinculacion/desvinculacion de cuentas, y evaluar riesgos de seguridad especificos de cada proveedor (scopes solicitados, manejo de tokens de proveedor).

## Errores comunes

- Solicitar mas scopes de los necesarios al proveedor OAuth, pidiendo permisos que la app no usa.
- No manejar el caso de un email que ya existe con otro metodo de login, generando cuentas duplicadas.
- Guardar tokens de acceso del proveedor externo sin necesidad, cuando solo se necesitaba el perfil una vez.
- No testear el flujo completo en staging antes de habilitarlo en produccion, dejando pasar bugs de callback URL o configuracion.

## Practica

- [ ] Registra una aplicacion de prueba en un proveedor OAuth (Google o GitHub).
- [ ] Implementa la strategy correspondiente en un proyecto Nest de practica.
- [ ] Implementa el manejo de "usuario ya existe con otro metodo de login".
- [ ] Verifica que el token emitido despues del login social sea compatible con el resto de tu sistema de auth.
- [ ] Anota que scopes minimos necesitarias pedir en tu proyecto real.

## Preguntas de repaso

- Por que OAuth2 es mas seguro que pedirle al usuario su contraseña de otro servicio?
- Que decision hay que tomar cuando el email de un login social ya existe con login tradicional?
- Que riesgo tiene solicitar scopes mas amplios de los necesarios?
- Como unificarias la sesion de un usuario que entro por login social con el resto de tu sistema de auth?
