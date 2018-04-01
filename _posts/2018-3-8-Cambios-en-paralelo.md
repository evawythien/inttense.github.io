---
layout: post
title: Cambios en paralelo
tags: design Eva programacion presentacion
comments: true
---

Hace unas semanas estuve en el ~WeCode~ en el taller que impartió <a href='http://www.eferro.net/' target='_blank'>eferro</a> sobre cambios paralelos. Este consistía en que nos proponia una serie de casos que debíamos solucionar siguiendo las siguientes reglas:

#### Reglas
* Nunca se debe de perder el servicio.
* Los despliegues requieren versiones en paralelo.
* Los despliegues deben de ser lo mas pequeños posibles.
* No se coordinan despliegues entre servicios.


### Escenario 1.
Tenemos la aplicación A, que utiliza el método calculo1 y visualiza el resultado. Pero queremos cambiar calculo1 por calculo2 (mismo interface).
Cáculo1 y calculo2 son complejos.


Ejemplo solución L1:
1. [app1 con calc1 + calc2 (sin conectar)]
2. [app1 -> abst1 -> calc1]
3. app1 -> abst1 -> calc1 y calc2. usa calc1 pero hace log si calc1 != calc2
4. [app1->abst1->calc2]
5. app1->calc2


### Escenario L2:
Tenemos la aplicación A, que usa calculo1 y visualiza el resultado. Queremos cambiar calculo1 por calculo2 (mismo interface). calculo1 y calculo2 complejos. calculo1 ejecuta sideeffect y necesitamos mantenerlo. sideeffect no se puede ejecutar dos veces.


### Escenario PE1:
Tenemos la app1, que usa calc1 y visualiza el resultado.
Queremos mejorar rendimiento calc1.



Lógica o performance: A tener en cuenta
Estrategia Branch By Abstraction
Instrumentación:
validar datos
medir performance


### Escenario P1:
Cambio nombre atributo member_status a membership en la entidad user (persistencia en tabla BD relacional).
Atributo sin relación a otras tablas
Cambio en código y en columna


### Escenario P2:

Transformación del atributo address a address, country en la entidad user Persistencia en tabla de base de datos relacional.
Los atributos no tienen relacion con otras tablas. Cambio en código y en columna.


### Escenario P3:

Cambio de tecnología persistencia para entidad User, resto de entidades solo se relacionan con user por el id.
User no se usa en joins.


Tras realizar estos ejercicios llegamos a la conclusión que para realizar cambios de la persistencia de nuestro software debemos de tener una serie de factores en cuenta:

Como el tiempo de migración, el volumen de datos, el TTL de datos, las operaciones idempotentes, los procesos re-arrancables, los scrips de validación de datos.


### Escenario C1
El servicio A envía,usando una cola, el mensaje1 con field_old al servicio B. field_old debe cambiar a field_new

### Escenario C2
El servicio A envía, usando una cola multicast,el mensaje1 con field_old al servicio B, C y D, field_old debe cambiar a field_new

### Escenario C3
El servicio A envía el evento1 (stream) con el campo field_old. Pero field_old debe cambiar en field_new. El servicio B puede leer eventos almacenados.
