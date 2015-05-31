---
layout: post
title: Concurrencia y paralelismo
---

Ultimamente leémos mucho sobre concurrencia y paralelismo, aunque están relacionadas son cosas diferentes.

Una forma de explicar ambos modelos puede ser la siguiente:

 - Concurrencia se utiliza cuando necesitas manejar eventos simultaneos. 

 - Paralelismo, por el contrario, es una solución para hacer el programa más rápido mediante el procesamiento de partes diferentes del programa al mismo tiempo(en paralelo).

¿Que tienen en común estos dos modelos?

Que ambos están mas alla de la programación tradicional, la secuencial, en el que las cosas suceden una despues de la otra.

>Mi esposa es profesora. Como la mayoría de los profesores, es una maestra de la multitarea. En un solo instante, ella está haciendo una taréa, pero ella está realizando muchas tareas concurrentemente. Mientras escucha un niño leer, podría tomarse el tiempo para calmar un aula ruidosa o responder a una pregunta. Esto es concurrente, pero no es paralelo(sólo es ella).
>
Si ella tiene un ayudante(uno de ellos escucha un lector, el otro responde preguntas), ahora tenemos algo que es concurrente y paralelo.
>
Imagina que la clase ha diseñado sus propias tarjetas de felicitación y quieren producirlas en masa. Una forma de hacerlo sería dar a cada niño la tarea de hacer cinco cartas. Esto es paralelo pero no(visto desde un nivel suficientemente alto) es concurrente sólo una tarea está en marcha.
> 
>> Seven Concurrency Models in Seven Weeks