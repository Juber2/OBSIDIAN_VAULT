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

# 01 - APT y Debian Ubuntu

## Que vas a aprender

En este capitulo vas a dominar **APT**, el gestor de paquetes de la familia Debian/Ubuntu, ya usado de forma introductoria en el capitulo de Fundamentos: repositorios, resolucion de dependencias, y comandos del dia a dia.

## Explicacion

APT gestiona software instalando paquetes `.deb` y resolviendo automaticamente sus **dependencias**: si instalas un paquete que necesita otra libreria, APT la instala tambien sin que tengas que buscarla manualmente. La lista de **repositorios** de donde APT descarga paquetes vive en `/etc/apt/sources.list` (y archivos en `/etc/apt/sources.list.d/`), y `apt update` refresca el indice local de paquetes disponibles en esos repositorios (sin instalar nada todavia, solo actualiza el catalogo).

Comandos clave: `apt install` instala un paquete, `apt remove` lo desinstala pero deja archivos de configuracion, `apt purge` lo elimina completamente incluyendo configuracion, `apt autoremove` limpia dependencias que ya no las necesita ningun paquete instalado, y `apt search`/`apt show` permiten explorar el catalogo antes de instalar algo.

## Modelo mental

Pensa APT como un asistente de compras muy meticuloso: cuando le pedis un producto especifico, revisa automaticamente que otros componentes necesita ese producto para funcionar, los agrega a la lista de compras sin que se lo pidas, y mantiene un catalogo actualizado de que hay disponible en las tiendas (repositorios) que le indicaste.

## Ejemplo base

```bash
sudo apt update                    # actualiza el catalogo local de paquetes disponibles
sudo apt upgrade                   # actualiza los paquetes instalados a sus ultimas versiones
sudo apt install nginx              # instala nginx y sus dependencias

apt search "servidor web"           # busca paquetes por palabra clave
apt show nginx                      # informacion detallada de un paquete

sudo apt remove nginx                # desinstala, deja configuracion
sudo apt purge nginx                 # desinstala completo, incluyendo configuracion
sudo apt autoremove                  # limpia dependencias huerfanas
```

## Como se usa en un proyecto real

1. Se ejecuta `apt update` antes de cualquier `apt install`, asegurando que se instala la version mas reciente disponible en los repositorios configurados.
2. Se usa `apt purge` en vez de `apt remove` cuando realmente se quiere eliminar todo rastro de un paquete, incluyendo su configuracion.
3. Se ejecuta `apt autoremove` periodicamente como parte de mantenimiento del sistema, limpiando dependencias que ya nadie usa.
4. Se documenta el proceso de instalacion de dependencias de un proyecto (que paquetes de sistema necesita, ademas de las dependencias de la aplicacion) para reproducibilidad entre servidores.
5. Se conecta este conocimiento con el capitulo de gestion de dependencias del libro de NestJS: APT resuelve dependencias del sistema operativo con el mismo principio que npm resuelve dependencias de una aplicacion Node.

## Criterio junior

Un junior debe poder actualizar el catalogo, instalar y desinstalar paquetes basicos con APT.

## Criterio mid

Un mid debe distinguir `remove` de `purge`, usar `autoremove` para mantenimiento, y buscar/explorar paquetes antes de instalarlos.

## Criterio senior

Un senior debe gestionar repositorios adicionales de forma segura (verificando su confiabilidad, similar al criterio visto para imagenes Docker), y documentar las dependencias de sistema de proyectos para reproducibilidad entre servidores.

## Errores comunes

- Instalar un paquete sin ejecutar `apt update` antes, obteniendo potencialmente una version desactualizada del catalogo local.
- Usar `apt remove` cuando en realidad se queria eliminar completamente la configuracion tambien (`apt purge`).
- No ejecutar `apt autoremove` periodicamente, acumulando dependencias huerfanas sin uso con el tiempo.
- Agregar repositorios de terceros sin verificar su confiabilidad, un riesgo similar al de usar imagenes Docker de fuentes no verificadas.

## Practica

- [ ] Actualiza el catalogo de paquetes y luego instala un paquete simple con APT.
- [ ] Busca informacion detallada de un paquete con `apt show` antes de instalarlo.
- [ ] Desinstala ese paquete con `apt purge` y verifica que su configuracion tambien se elimino.
- [ ] Ejecuta `apt autoremove` y observa que dependencias huerfanas limpia.
- [ ] Documenta las dependencias de sistema necesarias para un proyecto real tuyo.

## Preguntas de repaso

- Que hace `apt update` y por que no instala nada por si solo?
- Que diferencia hay entre `apt remove` y `apt purge`?
- Para que sirve `apt autoremove` en el mantenimiento regular de un sistema?
- Que riesgo tiene agregar repositorios de terceros sin verificar su confiabilidad?
