---
tags:
  - NestJS
  - TypeScript
  - Nest-DevOps
Creado: 2026-07-27
Relacionado:
  - DevOps Deploy
  - NESTJS
---

# 04 - Deploy en VPS y Cloud

## Que vas a aprender

En este capitulo vas a comparar las principales opciones de despliegue para una aplicacion NestJS: **VPS tradicional** con PM2 y reverse proxy, y opciones **cloud gestionadas** (contenedores en ECS/Cloud Run/App Service, o serverless), con sus tradeoffs de control vs simplicidad operativa.

## Explicacion

Desplegar en un **VPS** (con PM2 como process manager y Nginx como reverse proxy) da control total sobre el entorno, a costo de responsabilidad total: parches de seguridad del sistema operativo, configuracion de firewall, monitoreo manual, y escalamiento manual si el trafico crece. Es una opcion valida para proyectos chicos o con presupuesto ajustado.

Las opciones **cloud gestionadas** (contenedores administrados como ECS, Cloud Run, App Service) delegan gran parte de esa responsabilidad operativa al proveedor, con escalado automatico y menor superficie de mantenimiento, a cambio de menor control fino y potencialmente mayor costo a escala. NestJS tambien puede correr en modo **serverless** (adaptando el bootstrap para AWS Lambda), util para cargas esporadicas pero con la limitacion de cold starts y restricciones de duracion de ejecucion.

## Modelo mental

Pensa el VPS como ser dueño de una casa: control total, pero vos te encargas del mantenimiento. Pensa cloud gestionado como vivir en un edificio con administracion: menos control sobre detalles, pero el edificio se encarga del mantenimiento comun por vos.

## Ejemplo base

```bash
# VPS con PM2
pm2 start dist/main.js --name api -i max  # cluster mode, un proceso por core
pm2 save
pm2 startup

# Nginx como reverse proxy
# location / { proxy_pass http://localhost:3000; }
```

```typescript
// Adaptacion para AWS Lambda (serverless)
import { configure as serverlessExpress } from '@vendia/serverless-express';

export const handler = async (event, context) => {
  const app = await bootstrapServer();
  return serverlessExpress({ app: app.getHttpAdapter().getInstance() })(event, context);
};
```

## Como se usa en un proyecto real

1. Se elige VPS cuando el equipo tiene experiencia operando infraestructura y el presupuesto o los requisitos de control lo justifican.
2. Se elige cloud gestionado cuando se prioriza velocidad de entrega y menor carga operativa, aceptando el costo asociado.
3. En VPS, PM2 en modo cluster aprovecha todos los cores disponibles, reiniciando procesos caidos automaticamente.
4. En cualquier opcion, se separa la base de datos del servidor de aplicacion, para poder escalar y respaldar cada uno de forma independiente.
5. Se configura HTTPS (certificados automaticos con Let's Encrypt en VPS, o gestionado por el proveedor cloud) sin excepcion.

## Criterio junior

Un junior debe poder desplegar una aplicacion Nest simple en un VPS con PM2 y Nginx siguiendo una guia, o en una plataforma cloud gestionada basica.

## Criterio mid

Un mid debe configurar PM2 en modo cluster para aprovechar multiples cores, y comparar el costo real (no solo el precio de lista) entre VPS y opciones cloud gestionadas para un caso concreto.

## Criterio senior

Un senior debe decidir la estrategia de infraestructura del sistema completo segun el contexto del equipo (experiencia operando infraestructura, presupuesto, requisitos de escalamiento) y diseñar la arquitectura de despliegue con alta disponibilidad.

## Errores comunes

- Correr la aplicacion Node directamente con `node dist/main.js` sin un process manager, perdiendo reinicio automatico ante caidas.
- No configurar HTTPS, dejando trafico sin cifrar entre clientes y el servidor.
- Subestimar el trabajo operativo real de mantener un VPS (parches, seguridad, monitoreo) al comparar costos con opciones gestionadas.
- Mezclar base de datos y aplicacion en el mismo servidor sin plan de escalamiento independiente.

## Practica

- [ ] Despliega una aplicacion Nest de practica en un VPS con PM2 y Nginx como reverse proxy.
- [ ] Configura PM2 en modo cluster y verifica que use multiples cores.
- [ ] Compara el costo total (tiempo + dinero) de mantener esa app en VPS vs una opcion cloud gestionada.
- [ ] Configura HTTPS con un certificado gratuito.
- [ ] Anota que opcion elegirias para tu proyecto actual y por que.

## Preguntas de repaso

- Que ventaja de control da un VPS frente a una opcion cloud gestionada, y que costo operativo trae a cambio?
- Por que correr la app con un process manager como PM2 es mejor que correrla directo con `node`?
- Que limitacion tiene el modo serverless para una API con requests de larga duracion?
- Que factores considerarias antes de elegir entre VPS y cloud gestionado para un proyecto nuevo?
