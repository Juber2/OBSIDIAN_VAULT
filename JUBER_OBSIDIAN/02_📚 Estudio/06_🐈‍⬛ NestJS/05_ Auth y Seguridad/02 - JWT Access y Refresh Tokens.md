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

# 02 - JWT Access y Refresh Tokens

## Que vas a aprender

En este capitulo vas a aprender a implementar autenticacion basada en **JWT** con el patron de **access token + refresh token**, entendiendo por que un solo token de larga duracion es una mala practica de seguridad.

## Explicacion

Un JWT (JSON Web Token) es un token autocontenido, firmado, que el servidor puede validar sin consultar una base de datos (stateless). El problema de usar un unico token de larga duracion es que, si se filtra, sigue siendo valido hasta su expiracion sin forma de revocarlo facilmente.

El patron access + refresh resuelve esto: el **access token** vive poco tiempo (minutos) y se usa en cada request; el **refresh token** vive mas tiempo (dias) y se usa solo para pedir un nuevo access token cuando expira, generalmente guardado en un almacenamiento mas controlado (httpOnly cookie) y a veces registrado en base de datos para poder revocarlo.

## Modelo mental

Pensa el access token como una entrada de un dia a un evento: si se pierde, el daño es limitado porque expira pronto. El refresh token es como la tarjeta de socio que te permite sacar una nueva entrada sin hacer fila de nuevo, pero que el club puede desactivar si detecta un problema.

## Ejemplo base

```typescript
async login(user: User) {
  const payload = { sub: user.id, email: user.email };
  const accessToken = this.jwtService.sign(payload, { expiresIn: '15m' });
  const refreshToken = this.jwtService.sign(payload, {
    expiresIn: '7d',
    secret: process.env.JWT_REFRESH_SECRET,
  });
  await this.refreshTokensRepo.save({ userId: user.id, token: refreshToken });
  return { accessToken, refreshToken };
}

async refresh(token: string) {
  const stored = await this.refreshTokensRepo.findOne({ where: { token } });
  if (!stored) throw new UnauthorizedException();
  const payload = this.jwtService.verify(token, { secret: process.env.JWT_REFRESH_SECRET });
  return this.login({ id: payload.sub, email: payload.email } as User);
}
```

## Como se usa en un proyecto real

1. El access token se usa en el header `Authorization: Bearer` de cada request protegido.
2. El refresh token se guarda en un almacenamiento mas seguro (httpOnly cookie o almacenamiento seguro del cliente movil), nunca en `localStorage` si se puede evitar.
3. Se registran los refresh tokens emitidos en base de datos para poder revocarlos (logout real, deteccion de robo).
4. Se rota el refresh token en cada uso (refresh token rotation) para detectar reuso indebido si un token robado se usa despues del legitimo.
5. Se definen tiempos de expiracion segun el nivel de sensibilidad de la aplicacion (banca: minutos; app de consumo: horas).

## Criterio junior

Un junior debe poder generar y verificar un JWT basico, y entender la diferencia conceptual entre access y refresh token.

## Criterio mid

Un mid debe implementar el flujo completo de refresh, incluyendo el registro y validacion del refresh token contra base de datos para poder revocarlo.

## Criterio senior

Un senior debe diseñar la estrategia completa de sesiones del sistema (rotacion de refresh tokens, revocacion masiva ante incidente, expiracion segun sensibilidad) y evaluar tradeoffs entre JWT stateless y sesiones stateful segun el caso de uso.

## Errores comunes

- Usar un unico token de larga duracion sin mecanismo de revocacion.
- Guardar el refresh token en `localStorage`, exponiendolo a ataques XSS.
- No rotar el refresh token, dejando una ventana larga de uso si un token es robado.
- No tener forma de revocar tokens ante un incidente de seguridad (cuenta comprometida).

## Practica

- [ ] Implementa el flujo completo de login con access y refresh token.
- [ ] Implementa el endpoint de refresh que valida el refresh token contra base de datos.
- [ ] Implementa rotacion de refresh token en cada uso.
- [ ] Implementa un endpoint de logout que revoque el refresh token almacenado.
- [ ] Anota los tiempos de expiracion que usarias segun el tipo de aplicacion.

## Preguntas de repaso

- Por que un unico token de larga duracion es una mala practica de seguridad?
- Donde deberia guardarse el refresh token en el cliente y por que?
- Que es la rotacion de refresh tokens y que problema resuelve?
- Como implementarias un logout real con JWT stateless?
