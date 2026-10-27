---
tags:
  - Java
  - Seguridad
Creado: 2026-05-21
Actualizado: 2026-08-12
Relacionado:
  - Senior Java Engineer
  - Spring Security
---

# Seguridad Backend Java

Seguridad no es solo login. Incluye autorizacion, validacion, secretos, auditoria y manejo seguro de errores.

## OWASP Top 10 (2021) aplicado a un backend Java/Spring

El marco de referencia estándar de la industria — cuando en entrevista preguntan "¿conocés OWASP Top 10?", esto es lo que esperan que puedas mapear a código real, no solo recitar la lista.

| # | Riesgo OWASP | Qué significa en un backend Java | Mitigación concreta |
|---|---|---|---|
| A01 | **Broken Access Control** | Un endpoint no valida si el usuario puede acceder a *ese* recurso específico, solo si está autenticado | `@PreAuthorize` a nivel recurso (no solo rol), validar ownership explícito |
| A02 | **Cryptographic Failures** | Contraseñas en texto plano o con hash débil (MD5/SHA1), datos sensibles sin cifrar en tránsito/reposo | BCrypt/Argon2 para passwords, TLS obligatorio, cifrar campos sensibles en BD |
| A03 | **Injection** | SQL injection, JPQL injection, command injection | Prepared statements (JPA/Hibernate ya lo hace por default), nunca concatenar input del usuario en queries |
| A04 | **Insecure Design** | Falta de rate limiting en login, sin límite de intentos, lógica de negocio explotable | Rate limiting (ver [[04 - APIM y Application Gateway]]), threat modeling en el diseño, no solo en el código |
| A05 | **Security Misconfiguration** | CORS con `*` + credentials, Actuator expuesto sin protección, stack traces en producción | Ver [[05 - CORS y CSRF]] y [[07 - Actuator]] — configuración explícita por ambiente, nunca defaults inseguros en prod |
| A06 | **Vulnerable Components** | Dependencias con CVEs conocidos sin actualizar | `mvn dependency-check`, Dependabot/Snyk, mantener versiones al día |
| A07 | **Identification and Authentication Failures** | JWT sin expiración corta, sin invalidación posible, contraseñas débiles permitidas | Ver [[02 - JWT]] — expiración corta + refresh tokens, políticas de contraseña, MFA cuando aplique |
| A08 | **Software and Data Integrity Failures** | Deserializar JSON/objetos de fuentes no confiables sin validar, dependencias de CI/CD sin verificar integridad | Validar/sanitizar toda deserialización externa, firmar artifacts en el pipeline |
| A09 | **Security Logging and Monitoring Failures** | No hay registro de intentos de login fallidos, cambios de permisos, accesos a datos sensibles | Audit log de eventos de seguridad, alertas ante patrones anómalos (ver [[08 - Distributed Tracing]]) |
| A10 | **Server-Side Request Forgery (SSRF)** | El backend hace una request HTTP a una URL controlada por el usuario sin validar destino | Whitelist de dominios permitidos, no seguir redirects ciegamente, aislar de red interna |

## Riesgos comunes (resumen operativo)

| Riesgo | Prevencion |
|---|---|
| Broken access control | Validar permisos por recurso |
| Injection | Prepared statements, validacion |
| Secret leakage | Secrets manager, no logs sensibles |
| JWT debil | Expiracion, firma fuerte, audience |
| Error leakage | Respuestas genericas, logs internos |

## Autorizacion por recurso

No basta con `ROLE_ADMIN` o `ROLE_USER`. Pregunta si el usuario puede acceder a ese recurso especifico.

```java
if (!currentUser.canAccess(order)) {
    throw new AccessDeniedException("Forbidden");
}
```

Esto es exactamente **A01 (Broken Access Control)** — el riesgo #1 del ranking actual, y el más común en aplicaciones reales según los propios reportes de OWASP.

## Secrets: nunca en el código ni en logs

```java
// ❌ Nunca
log.info("Token generado: {}", jwt);
log.info("Password del usuario: {}", password);

// ✅ Loguear solo lo necesario para debuggear sin exponer el secreto
log.info("Token generado para usuario: {}", usuario.getEmail());
```

Ver [[02 - AKS Fundamentos]] y [[04 - ConfigMaps Secrets y Volumenes]] para cómo se gestionan secretos en producción sin hardcodearlos (Azure Key Vault, Kubernetes Secrets con vault externo).

## Dependency scanning — A06 en la práctica

```bash
# Maven — detectar CVEs en dependencias
mvn org.owasp:dependency-check-maven:check
```

Integrarlo al pipeline de CI (ver [[GitHub Actions CI CD]]) para que un CVE crítico bloquee el merge, no que se descubra meses después en un audit.

## Checklist
- [ ] Valido autenticacion y autorizacion.
- [ ] No confio en ids enviados por cliente.
- [ ] No logueo tokens ni passwords.
- [ ] Manejo errores sin revelar detalles internos.
- [ ] Mantengo dependencias actualizadas.
- [ ] Puedo nombrar y explicar al menos 6 de los 10 riesgos OWASP con un ejemplo concreto en Java/Spring.
- [ ] Tengo dependency scanning en el pipeline de CI, no solo "de vez en cuando manual".
- [ ] Sé qué es SSRF y por qué una simple validación de URL de entrada no siempre alcanza para prevenirlo.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

