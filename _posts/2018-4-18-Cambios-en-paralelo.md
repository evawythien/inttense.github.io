---
layout: post
title: Cambios en paralelo
tags: agile development
comments: true
---

Hace unas semanas estuve en el ~WeCode~ en el taller que impartió <a href='http://www.eferro.net/' target='_blank'>eferro</a> sobre cambios paralelos. Este consistía en que nos proponia una serie de casos que debíamos solucionar siguiendo las siguientes reglas:

#### Reglas:
* Nunca se debe de perder el servicio.
* Los despliegues requieren versiones en paralelo.
* Los despliegues deben de ser lo mas pequeños posibles.
* No se coordinan despliegues entre servicios.


#### Escenario 1.1.
Tenemos la aplicación A, que utiliza el método ```calculo1``` y visualiza el resultado. Pero queremos cambiar ```calculo1``` por ```calculo2``` (mismo interface).
```Cáculo1``` y ```calculo2``` son complejos.

Ejemplo solución escenario 1:
1. Añadimos a la applicación ```calculo2```, pero no realizamos ninguna llamada.
2. Abstraemos ```calculo1``` y ```calculo2```.
3. Ejecutamos ```calculo1``` y ```calculo2```. Usamos ```calculo1```, pero loggeamos si ```calculo1 != calculo2```.
4. Utilizamos solo ```calculo2```.

#### Escenario 1.2:
Tenemos la aplicación A, que usa ```calculo1``` y visualiza el resultado. Queremos cambiar ```calculo1``` por calculo2 (mismo interface). ```calculo1``` y ```calculo2``` complejos. calculo1 ejecuta ```sideeffect``` y necesitamos mantenerlo. ```sideeffect``` no se puede ejecutar dos veces.

Solución:
1. Añadimos a la applicación ```calculo2```, pero no realizamos ninguna llamada.
2. Abstraemos sideeffect fuera de los métodos.
3. Abstraemos ```calculo1```.
4. Ejecutamos ```calculo1``` y ```calculo2```. Usamos ```calculo1```, pero loggeamos si ```calculo1 != calculo2```.
5. Abstraemos ```calculo2```.
6. Utilizamos solo ```calculo2```.

#### Escenario 1.3:
Tenemos la applicación A, que usa ```calculo1``` y visualiza el resultado. Queremos mejorar su rendimiento.

Solución:
1. Duplicamos el método calculo1 con otro nombre.
2. Abstraemos ```calculo1```.
3. Refactorizamos el nuevo método.
4. Ejecutamos los dos métodos. Usamos ```calculo1```, pero loggeamos si el resultado de los dos métodos son diferentes.
5. Utilizamos el método refactorizado.

El método que hemos decidido utilizar para realizar esta serie de cambios es la estrategia llamada Branch By Abstraction.

> Branch By Abstraction es una técnica que se utiliza para realizar un cambio a gran escala pero de forma gradual en un sistema de software. Lo que permite liberar el sistema regularmente mientras el cambio aún está en progreso.

Comenzamos con una situación en la que varias partes del sistema de software tienen algún tipo de dependencia. Gradualmente vamos moviendo todo el código de cliente para utilizar la capa de abstracción hasta que la capa de abstracción realiza toda la interacción. Al hacer esto, aprovechamos la oportunidad para mejorar la cobertura de pruebas unitarias.

Construimos un nuevo proveedor que implementa las características requeridas por una parte del código del cliente utilizando la misma capa de abstracción. Una vez que estamos listos, cambiamos esa sección del código del cliente para usar el nuevo proveedor.

#### Escenario 2.1:
Queremos realizar un cambio en el nombre del atributo ```member_status``` a ```membership``` en la entidad user (persistencia en tabla BD relacional). *NOTA: El atributo no tiene relación con otras tablas. El cambio hay que realizarlo en el código y en la columna.*

Solución:
1. Creamos el nuevo atributo ```membership``` y empezamos también a insertar datos. Pero seguimos leyendo de ```member_status```.
2. Introducimos lógica, si ```membership``` está vacío leermos de ```member_status```.
3. Pasamos los datos que quedan al nuevo campo.
4. Empezamos a leer solo de ```membership```.

#### Escenario 2.2:

Transformación del atributo ```address``` a ```address``` y ```country``` en la entidad User. Anteriormente se escribía la dirección con el formarto "calle, ciudad".
La persistencia se realiza en una tabla de base de datos relacional. *NOTA: El atributo no tiene relación con otras tablas. El cambio hay que realizarlo en el código y en la columna.*

Solución:
1. Creamos el nuevo atributo ```country``` y empezamos a insertar datos.
2. Leemos los datos del nuevo atributo ```country```, si no hay nada leemos del campo antiguo ```address```. (Mediante lógica)
3. Escribimos en el campo ```address``` los datos sin el pais.
4. Actualizar los datos antiguos al nuevo formato.
5. Eliminamos la lógica del código.

#### Escenario 2.3:

Cambio de tecnología de persistencia para entidad User, el resto de entidades solo se relacionan con user por el id.
*NOTA: La entidad User no se usa en joins.*

Solución:

Suponemos por ejemplo que el cambio se va a hacer de una base de datos Sql, a NoSql.
1. Creamos la bbdd NoSql.
2. Hacemos que nuestra aplicación lea primero de la bbdd nueva, y luego de la vieja.
3. Empezamos a escribir en la bbdd nueva.

> Tras realizar estos ejercicios llegamos a la conclusión que para realizar cambios de la persistencia de nuestro software debemos de tener una serie de factores en cuenta:
como el tiempo de migración, el volumen de datos, el TTL de datos, las operaciones idempotentes, los procesos re-arrancables, los scrips de validación de datos..

#### Escenario 3.1:
El servicio A envía, usando una cola, el mensaje1 con ```field_old``` al servicio B. ```field_old``` debe cambiar a ```field_new```.

Solución:
1. El servicio A envía los dos campos, ```field_old``` y ```field_new```, al servicio B.
2. El servicio B, solo lee el campo ```field_new```.
3. El servicio A, solo envía el campo ```field_new```.

#### Escenario 3.2:
El servicio A envía, usando una cola multicast, el mensaje1 con ```field_old``` al servicio B, C y D, ```field_old``` debe cambiar a ```field_new```.

Solución:
1. Hacemos que el servicio envíe los campos, ```field_old``` y ```field_new```.
2. Hacemos que el servicio B solamente lea el campo ```field_new```.
3. Hacemos que el servicio C solamente lea el campo ```field_new```.
4. Hacemos que el servicio D solamente lea el campo ```field_new```.
5. El servicio A, solo envía el campo ```field_new```.

> Para solucionar estos esacenarios se ha aplicado el principio de robustez, también conocido como Postel's law. Este principio recomienda a los programadores que asuman que la red está llena de entidades malévolas que envían paquetes diseñados para tener el peor efecto posible.

*"Be conservative in what you do, be liberal in what you accept from others."*

Por esta razón los protocolos deben permitir la adición de nuevas funcionalidades para campos ya existentes en futuras versiones mediante la aceptación de mensajes desconocidos. De esta manera se debe evitar enviar mensajes que puedan exponer deficiencias en los receptores, y diseñar su código no solo para sobrevivir, sino también para limitar la cantidad de interrupciones que tales hosts pueden causar y así facilidad de comunicación.

Relizar estos ejercicios ha supuesto para mi una manera diferente de afrontar una serie de problemas, ya que cada cambio que aplicaba tenía que ser lo mas pequeño posible para no perjudicar la estabilidad el software en ningún momento.

<h5 class="C">¿Resolverías algún ejercicio de otra manera?</h5>
<h5 class="C">¡Coméntalo aquí abajo!</h5>
