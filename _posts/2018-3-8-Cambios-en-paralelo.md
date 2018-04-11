---
layout: post
title: Cambios en paralelo
tags: Agile Development
comments: true
---

Hace unas semanas estuve en el ~WeCode~ en el taller que impartió <a href='http://www.eferro.net/' target='_blank'>eferro</a> sobre cambios paralelos. Este consistía en que nos proponia una serie de casos que debíamos solucionar siguiendo las siguientes reglas:

### Reglas
* Nunca se debe de perder el servicio.
* Los despliegues requieren versiones en paralelo.
* Los despliegues deben de ser lo mas pequeños posibles.
* No se coordinan despliegues entre servicios.


#### Escenario 1.
Tenemos la aplicación A, que utiliza el método calculo1 y visualiza el resultado. Pero queremos cambiar calculo1 por calculo2 (mismo interface).
Cáculo1 y calculo2 son complejos.

Ejemplo solución L1:
1. Añadimos a la applicación calculo2, pero no realizamos ninguna llamada.
2. [app1 -> abst1 -> calc1]
3. Ejecutamos calculo1 y calculo2. Usamos calculo1, pero loggeamos si calculo1 != calculo2.
4. [app1->abst1->calc2]
5. Utilizamos solo calculo2.


#### Escenario L2:
Tenemos la aplicación A, que usa calculo1 y visualiza el resultado. Queremos cambiar calculo1 por calculo2 (mismo interface). calculo1 y calculo2 complejos. calculo1 ejecuta sideeffect y necesitamos mantenerlo. sideeffect no se puede ejecutar dos veces.


Solución:
1. Añadimos a la applicación calculo2, pero no realizamos ninguna llamada.
2. Abstraemos sideeffect fuera de los métodos.
3. [app1 -> abst1 -> calc1]
4.Ejecutamos calculo1 y calculo2. Usamos calculo1, pero loggeamos si calculo1 != calculo2.
5. [app1->abst1->calc2]
6. Utilizamos solo calculo2.

### Escenario PE1:
Tenemos la applicación A, que usa calculo1 y visualiza el resultado. Queremos mejorar su rendimiento.

Solución:
1.

> Lógica o performance: Para realizar esta serie de cambios debemos de tener en cuenta
Estrategia Branch By Abstraction
Instrumentación:
validar datos
medir performance


#### Escenario P1:
Queremos realizar un cambio en el nombre del atributo member_status a membership en la entidad user (persistencia en tabla BD relacional).

NOTA: El atributo no tiene relación con otras tablas. El cambio hay que realizarlo en el código y en la columna.

Solución:
1. Creamos el nuevo atributo membership y empezamos también a insertar datos. Pero seguimos leyendo de member_status.
2. Introducimos lógica, si membership está vacío leermos de member_status.
3. Pasamos los datos que quedan al nuevo campo.
4. Empezamos a leer solo de membership.

#### Escenario P2:

Transformación del atributo address a address y country en la entidad User. Anteriormente se escribía la dirección con el formarto "calle, ciudad".
La persistencia se realiza en una tabla de base de datos relacional.

NOTA: El atributo no tiene relación con otras tablas. El cambio hay que realizarlo en el código y en la columna.

Solución:
1. Creamos el nuevo atributo country y empezamos a insertar datos.
2. Leemos los datos del nuevo atributo country, si no hay nada leemos del campo antiguo address. (Mediante lógica)
3. Escribimos en el campo address los datos sin el pais.
4. Actualizar los datos antiguos al nuevo formato.
5. Eliminamos la lógica del código.

#### Escenario P3:

Cambio de tecnología de persistencia para entidad User, el resto de entidades solo se relacionan con user por el id.
NOTA: La entidad User no se usa en joins.

Solución:

Suponemos por ejemplo que el cambio se va a hacer de una base de datos Sql, a NoSql.
1. Creamos la bbdd NoSql.
2. Hacemos que nuestra aplicación lea primero de la bbdd nueva, y luego de la vieja.
3. Empezamos a escribir en la bbdd nueva.


> Tras realizar estos ejercicios llegamos a la conclusión que para realizar cambios de la persistencia de nuestro software debemos de tener una serie de factores en cuenta:
Como el tiempo de migración, el volumen de datos, el TTL de datos, las operaciones idempotentes, los procesos re-arrancables, los scrips de validación de datos..


### Escenario C1
El servicio A, envía usando una cola, el mensaje1 con field_old al servicio B. field_old debe cambiar a field_new.

Solución:
1. El servicio A envía los dos campos, field_old y field_new, al servicio B.
2. El servicio B, solo lee el campo field_new.
3. El servicio A, solo envía el campo field_new.

#### Escenario C2
El servicio A envía, usando una cola multicast,el mensaje1 con field_old al servicio B, C y D, field_old debe cambiar a field_new.

Solución:
1. Hacemos que el servicio envíe los campos, field_old y field_new.
2. Hacemos que el servicio B solamente lea el campo field_new.
3. Hacemos que el servicio C solamente lea el campo field_new.
4. Hacemos que el servicio D solamente lea el campo field_new.
5. El servicio A, solo envía el campo field_new.

#### Escenario C3
El servicio A envía el evento1 (stream) con el campo field_old. Pero field_old debe cambiar en field_new. El servicio B puede leer eventos almacenados.
