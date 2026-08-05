---
tags:
  - SQLServer
  - Proyectos
  - SQLServer-Proyectos
Creado: 2026-07-29
Relacionado:
  - Proyectos
  - SQL SERVER
---

# 03 - Proyecto Always On Basico

## Que vas a aprender

En este proyecto vas a aplicar el capitulo de Alta Disponibilidad configurando un Availability Group basico entre dos contenedores de SQL Server en Docker, aplicando directamente lo aprendido en el capitulo de contenedores.

## Explicacion

Configurar Always On AG completo requiere normalmente un Windows Server Failover Cluster, pero para fines de aprendizaje es posible simular una configuracion basica de dos nodos usando contenedores Docker en la misma red, conectando dos instancias de SQL Server (imagen oficial ya usada en el capitulo de contenedores) como primaria y secundaria, con `synchronous commit` y failover manual (sin la complejidad completa del cluster de Windows, apropiado para practicar el concepto sin la infraestructura de produccion completa).

El objetivo de este proyecto no es replicar exactamente una configuracion de produccion (que requeriria WSFC real), sino entender practicamente los pasos del proceso: crear el endpoint de espejeo, unir la replica secundaria, verificar el estado de sincronizacion con las DMVs correspondientes, y practicar un failover manual observando el cambio de roles entre primaria y secundaria.

## Modelo mental

Pensa este proyecto como armar una maqueta a escala de un sistema de alta disponibilidad real: no tiene toda la complejidad de la version de produccion completa (el cluster de Windows real), pero te permite practicar y entender el flujo conceptual completo en un ambiente controlado y descartable.

## Ejemplo base

```bash
# Dos contenedores de SQL Server en la misma red Docker
docker network create ag-network

docker run -d --name sqlnode1 --network ag-network \
    -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=ContraseñaSegura123!" \
    -p 1433:1433 mcr.microsoft.com/mssql/server:2022-latest

docker run -d --name sqlnode2 --network ag-network \
    -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=ContraseñaSegura123!" \
    -p 1434:1433 mcr.microsoft.com/mssql/server:2022-latest
```

```sql
-- Verificar el estado de sincronizacion (una vez configurado el AG)
SELECT ag.name, drs.synchronization_state_desc, drs.is_local
FROM sys.dm_hadr_database_replica_states drs
JOIN sys.availability_groups ag ON drs.group_id = ag.group_id;
```

## Como se usa en un proyecto real

1. Se levantan dos contenedores de SQL Server en la misma red Docker, simulando dos nodos separados.
2. Se configura el endpoint de espejeo y se habilita Always On AG en ambas instancias (requiere habilitar la feature en el contenedor).
3. Se crea un Availability Group basico uniendo ambos nodos con `synchronous commit`.
4. Se verifica el estado de sincronizacion consultando las DMVs correspondientes, confirmando que los datos se replican correctamente.
5. Se practica un failover manual, observando como el rol de primaria/secundaria cambia entre los nodos.

## Criterio de exito

- Configuraste exitosamente dos instancias de SQL Server en contenedores conectadas en un Availability Group basico.
- Verificaste el estado de sincronizacion usando `sys.dm_hadr_database_replica_states`.
- Practicaste un failover manual y confirmaste el cambio de roles entre primaria y secundaria.
- Documentaste el proceso completo paso a paso como si fuera una guia de referencia futura.

## Errores comunes

- Subestimar la configuracion de red necesaria entre contenedores para que el endpoint de espejeo funcione correctamente.
- No verificar el estado de sincronizacion antes de intentar un failover, arriesgando perdida de datos si la replica secundaria no estaba al dia.
- Confundir esta configuracion de practica simplificada con una configuracion de produccion real, que requeriria WSFC completo.
- No documentar el proceso paso a paso, perdiendo la oportunidad de tener una referencia futura propia.

## Practica

- [ ] Levanta dos contenedores de SQL Server en la misma red Docker.
- [ ] Configura un Availability Group basico entre ambos nodos.
- [ ] Verifica el estado de sincronizacion con las DMVs correspondientes.
- [ ] Practica un failover manual y observa el cambio de roles.
- [ ] Documenta el proceso completo como una guia de referencia paso a paso.

## Preguntas de repaso

- Que diferencia hay entre esta configuracion de practica y un Always On AG de produccion real?
- Que verificaste con `sys.dm_hadr_database_replica_states` antes de considerar el AG funcional?
- Que observaste durante el failover manual respecto al cambio de roles?
- Que limitaciones tiene esta configuracion simplificada que un ambiente de produccion real no tendria?
