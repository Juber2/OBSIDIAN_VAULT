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

# 03 - Compilacion desde Fuente

## Que vas a aprender

En este capitulo vas a aprender a compilar software desde codigo fuente cuando no existe un paquete disponible en los repositorios estandar: el flujo clasico `./configure && make && make install`, y las herramientas de compilacion necesarias.

## Explicacion

Cuando un software no esta disponible como paquete `.deb`/`.rpm`, o se necesita una version/configuracion especifica no ofrecida por el paquete estandar, la alternativa es compilar desde el codigo fuente. El flujo clasico en proyectos C/C++ es: `./configure` (verifica el sistema y genera un Makefile adaptado, permitiendo pasar opciones de compilacion), `make` (compila el codigo fuente en binarios usando el Makefile generado), y `make install` (copia los binarios compilados a su ubicacion final en el sistema, tipicamente `/usr/local/`).

Antes de poder compilar, se necesitan herramientas de compilacion (`build-essential` en Debian/Ubuntu, `Development Tools` group en RedHat/derivados) que incluyen `gcc`, `make`, y otras utilidades. Un riesgo importante de esta via es que el software compilado manualmente **no se gestiona** por APT/DNF: no aparece en actualizaciones automaticas, y desinstalarlo require hacerlo manualmente (o usando `make uninstall` si el proyecto lo soporta), a diferencia de paquetes gestionados que se actualizan y desinstalan de forma centralizada.

## Modelo mental

Pensa instalar un paquete gestionado como comprar un mueble ya armado de una tienda de confianza, con garantia y soporte postventa organizado (actualizaciones automaticas). Compilar desde fuente es como comprar la madera y las herramientas y construir el mueble vos mismo: te da control total sobre el resultado, pero perdes toda la infraestructura de mantenimiento organizado que viene con el producto de tienda.

## Ejemplo base

```bash
sudo apt install build-essential           # herramientas de compilacion basicas (Debian/Ubuntu)
sudo dnf groupinstall "Development Tools"   # equivalente en RedHat/derivados

wget https://ejemplo.com/software-1.0.tar.gz
tar xzf software-1.0.tar.gz
cd software-1.0

./configure --prefix=/usr/local            # verifica el sistema, genera Makefile
make                                        # compila
sudo make install                            # instala los binarios compilados
```

## Como se usa en un proyecto real

1. Se prioriza siempre instalar desde el gestor de paquetes del sistema (APT/DNF) sobre compilar desde fuente, dado que da actualizaciones y desinstalacion gestionadas automaticamente.
2. Se compila desde fuente solo cuando realmente no existe alternativa: version especifica no disponible como paquete, opciones de compilacion custom necesarias, o software muy nuevo aun no empaquetado.
3. Se documenta claramente cualquier software compilado manualmente en un servidor, incluyendo version y opciones usadas, dado que no aparece en la gestion centralizada de paquetes del sistema.
4. Se instala en `/usr/local/` (el prefijo convencional para software instalado manualmente, ya visto en el capitulo de FHS del libro), separandolo claramente del software gestionado por el sistema en `/usr/`.
5. Se evalua siempre migrar a una version empaquetada oficialmente en cuanto este disponible, reduciendo la carga de mantenimiento manual a largo plazo.

## Criterio junior

Un junior debe poder seguir el flujo `./configure && make && make install` para compilar un software simple desde codigo fuente.

## Criterio mid

Un mid debe evaluar cuando compilar desde fuente es realmente necesario versus usar el gestor de paquetes, y documentar software compilado manualmente para mantenimiento futuro.

## Criterio senior

Un senior debe minimizar el uso de software compilado manualmente en produccion (favoreciendo paquetes gestionados o containers, ver libro de Docker), y establecer el proceso de documentacion obligatoria para cualquier excepcion justificada.

## Errores comunes

- Compilar desde fuente por defecto sin verificar primero si existe un paquete gestionado equivalente en los repositorios del sistema.
- No documentar software compilado manualmente, generando servidores con software "misterioso" que nadie recuerda como se instalo ni actualizo.
- No instalar en `/usr/local/` (o una ubicacion clara separada), mezclando software compilado manualmente con archivos gestionados por el sistema de paquetes.
- No planificar la actualizacion futura de software compilado manualmente, dejandolo desactualizado indefinidamente sin proceso de mantenimiento.

## Practica

- [ ] Instala las herramientas de compilacion basicas de tu distribucion (`build-essential` o `Development Tools`).
- [ ] Compila un software simple desde fuente siguiendo el flujo `./configure && make && make install`.
- [ ] Verifica donde quedo instalado (deberia ser en `/usr/local/` por defecto).
- [ ] Documenta ese software compilado manualmente: version, opciones usadas, fecha.
- [ ] Anota un criterio propio para decidir cuando compilar desde fuente es realmente necesario.

## Preguntas de repaso

- Que hace cada paso del flujo `./configure && make && make install`?
- Por que el software compilado manualmente no recibe actualizaciones automaticas como un paquete gestionado?
- Donde convencionalmente se instala software compilado manualmente, y por que esa separacion es importante?
- Cuando estaria realmente justificado compilar desde fuente en vez de usar el gestor de paquetes del sistema?
