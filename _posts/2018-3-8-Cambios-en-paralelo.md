---
layout: post
title: Cambios en paralelo
tags: design
comments: true
---

Introduccion aqui. Introduccion aqui. Introduccion aqui. Introduccion aqui.Introduccion aqui.Introduccion aqui.Introduccion aqui.Introduccion aqui.Introduccion aqui. Introduccion aqui. Introduccion aqui.Introduccion aqui.Introduccion aqui.Introduccion aqui. Introduccion aqui.Introduccion aqui.Introduccion aqui.Introduccion aqui.Introduccion aqui.Introduccion aqui.Introduccion aqui.Introduccion aqui.


Reglas
Nunca se debe de perder el servicio.
Los despliegues requieren versiones en paralelo.
Los despliegues deben de ser lo mas pequeños posibles.
No se coordinan despliegues entre servicios.


# Escenario 1.
Tenemos la aplicación A, que utiliza el método calculo1 y visualiza el resultado. Pero queremos cambiar calculo1 por calculo2 (mismo interface).
Cáculo1 y calculo2 son complejos.


Ejemplo solución L1:
[app1 con calc1 + calc2 (sin conectar)]
[app1 -> abst1 -> calc1]
app1 -> abst1 -> calc1 y calc2. usa calc1 pero hace log si calc1 != calc2
[app1->abst1->calc2]
app1->calc2


Escenario L2:
Tenemos la app1, que usa calc1 y visualiza el resultado. Queremos cambiar calc1 por calc2 (mismo interface). calc1 y calc2 complejos. calc1 ejecuta sideeffect y necesitamos mantenerlo. sideeffect no se puede ejecutar dos veces.


Escenario PE1:
Tenemos la app1, que usa calc1 y visualiza el resultado.
Queremos mejorar rendimiento calc1.



Lógica o performance: A tener en cuenta
Estrategia Branch By Abstraction
Instrumentación:
validar datos
medir performance


Escenario P1:
Cambio nombre atributo member_status a membership en la entidad user (persistencia en tabla BD relacional).
Atributo sin relación a otras tablas
Cambio en código y en columna
