---
tags:
  - Linux
  - Paquetes
  - Linux-Paquetes
Creado: 2026-07-28
Relacionado:
  - Gestion de Paquetes y Software
  - LINUX
---

# 05 - Repositorios y Fuentes de Confianza

## Que vas a aprender

En este capitulo vas a consolidar el criterio de seguridad para agregar **repositorios de terceros** al sistema: verificacion de firmas GPG, riesgos de cadena de suministro (conectando con lo visto en el libro de Docker), y buenas practicas antes de confiar en una fuente nueva.

## Explicacion

Cada repositorio de paquetes firma sus paquetes con una **clave GPG**, que el gestor de paquetes verifica antes de instalar, garantizando que el paquete descargado realmente proviene de esa fuente y no fue alterado en transito. Agregar un repositorio de terceros implica agregar su clave GPG como confiable (`apt-key`, o el metodo moderno con archivos en `/etc/apt/keyrings/`), lo cual es efectivamente darle a esa fuente permiso de instalar software con privilegios de root en tu sistema.

Este es exactamente el mismo riesgo de **cadena de suministro de software** visto en el capitulo de Seguridad del libro de Docker (imagenes de fuentes confiables): un repositorio de terceros comprometido, o directamente malicioso desde el inicio, puede distribuir software con backdoors o malware, y el sistema lo instalaria confiando ciegamente en la firma si la clave ya fue agregada como confiable.

## Modelo mental

Pensa agregar un repositorio de terceros como darle a alguien una copia de la llave de tu casa: confias en que va a usarla solo para las razones que le diste, pero tecnicamente ahora tiene acceso completo. Verificar la firma GPG es como comprobar que la persona que toca la puerta realmente es quien dice ser (usando esa llave que le diste) antes de dejarla entrar, no simplemente confiar en la palabra de cualquiera.

## Ejemplo base

```bash
# Metodo moderno recomendado: clave GPG en un keyring dedicado, no globalmente confiable
curl -fsSL https://ejemplo.com/gpg-key.asc | sudo gpg --dearmor -o /etc/apt/keyrings/ejemplo.gpg
echo "deb [signed-by=/etc/apt/keyrings/ejemplo.gpg] https://repo.ejemplo.com/ stable main" | \
  sudo tee /etc/apt/sources.list.d/ejemplo.list

sudo apt update
apt-cache policy nombre-paquete   # verifica desde que repositorio se instalaria un paquete
```

## Como se usa en un proyecto real

1. Se verifica la reputacion y el origen oficial de un repositorio antes de agregarlo (documentacion oficial del proyecto, no un blog o foro aleatorio con instrucciones copiadas).
2. Se usa el metodo moderno de claves GPG especificas por repositorio (`signed-by` con un keyring dedicado) en vez del metodo antiguo `apt-key add`, que agregaba la clave como confiable globalmente para todos los repositorios.
3. Se documenta cada repositorio de terceros agregado a un servidor, con su proposito y fuente oficial, como parte de la documentacion de infraestructura.
4. Se audita periodicamente los repositorios configurados en servidores de produccion (`/etc/apt/sources.list.d/`), removiendo los que ya no son necesarios.
5. Se aplica el mismo criterio de confianza visto para imagenes Docker: preferir fuentes oficiales verificadas sobre repositorios de terceros no verificados, salvo necesidad real y justificada.

## Criterio junior

Un junior debe entender que agregar un repositorio de terceros implica confiar en esa fuente para instalar software con privilegios de root, y verificar la firma GPG antes de confiar.

## Criterio mid

Un mid debe usar el metodo moderno de claves GPG especificas por repositorio, y verificar la fuente oficial de un repositorio antes de agregarlo a un servidor.

## Criterio senior

Un senior debe auditar periodicamente los repositorios configurados en toda la infraestructura, estableciendo una politica de aprobacion para nuevos repositorios de terceros, y aplicando el mismo rigor de cadena de suministro visto para imagenes Docker.

## Errores comunes

- Agregar un repositorio de terceros copiando instrucciones de un blog o foro sin verificar que realmente proviene de la fuente oficial del proyecto.
- Usar el metodo antiguo `apt-key add` que agrega una clave como confiable globalmente, en vez del metodo moderno con `signed-by` limitado a un repositorio especifico.
- No auditar repositorios configurados en servidores de produccion con el tiempo, dejando fuentes obsoletas o innecesarias acumuladas.
- No aplicar el mismo criterio de confianza usado para imagenes Docker al evaluar repositorios de paquetes del sistema operativo.

## Practica

- [ ] Agrega un repositorio de terceros de una herramienta legitima usando el metodo moderno de claves GPG con `signed-by`.
- [ ] Verifica con `apt-cache policy` desde que repositorio se instalaria un paquete especifico.
- [ ] Audita los repositorios configurados en un sistema real o de practica.
- [ ] Investiga la diferencia entre el metodo antiguo `apt-key add` y el moderno con keyrings especificos.
- [ ] Documenta el proceso de aprobacion que aplicarias antes de agregar un nuevo repositorio de terceros en tu equipo.

## Preguntas de repaso

- Que garantiza la verificacion de firma GPG de un paquete antes de instalarlo?
- Por que el metodo moderno de `signed-by` con keyrings especificos es mas seguro que `apt-key add` global?
- Que riesgo de cadena de suministro comparte agregar un repositorio de terceros con usar una imagen Docker no verificada?
- Como auditarias periodicamente los repositorios configurados en un servidor de produccion?
