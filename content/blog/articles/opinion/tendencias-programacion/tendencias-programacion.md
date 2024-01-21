+++
title = "Tendencias en la programación"
date = "2019-10-11"
description = "Opinión sobre las últimas tendencias en programación"
thumbnail = "/articles/opinion/tendencias-programacion/tendencias-programacion.jpg"
tags = [ "Opinión" ]
+++

No sé si recuerdan cómo surgieron las metodologías ágiles. No se preocupen, este artículo no va de eso; simplemente recordemos
que el concepto nace como contraposición a las metodologías como CMMI que anteponían la gestión de los requisitos
al desarrollo del software.

Por situarnos, la metodología utilizada como contraejemplo era la metodología en cascada. Decían algo así como 
"La metodología en cascada es el demonio, nadie en su sano juicio debería utilizar la metodología en cascada", no exactamente
así, pero entendéis la idea.

Recuerdo que en mi primer año de carrera, en la asignatura de Informática Básica, una de nuestras lecturas recomendadas era
[Managing the development of large Software systems](http://www-scf.usc.edu/~csci201/lectures/Lecture11/royce1970.pdf) 
del Dr Winston W. Royce publicado en 1.970.
	
En ese artículo, se define la metodología en cascada prácticamente en su segunda página con este gráfico que ha pasado a la posteridad:

![Metodología en cascada](/blog/articles/opinion/tendencias-programacion/metodologia-en-cascada.jpg "Definición de la metodología en cascada")

Sé que esto ya lo sabíais, pero antes que saltéis directamente a otra página, permitidme que os muestre una copia del párrafo
inmediatemente posterior a ese gráfico (las negritas son mías):

> I believe in this concept, but the implementation described above is **risky and invites failure**. The
> problem is illustrated in Figure 4. The testing phase which occurs at the end of the development cycle is the
> first event for which timing, storage, input/output transfers, etc., are experienced as distinguished from
> analyzed. These phenomena are not precisely analyzable. They are not the solutions to the standard partial
> differential equations of mathematical physics for instance. **Yet if these phenomena fail to satisfy the various
> external constraints, then invariably a major redesign is required.** A simple octal patch or redo of some isolated
> code will not fix these kinds of difficulties. The required design changes are likely to be so disruptive that the
> software requirements upon which the design is based and which provides the rationale for everything are
> violated. **Either the requirements must be modified, or a substantial change in the design is required**. In effect
> the development process has returned to the origin and one can expect up to a l00-percent overrun in schedule
> and/or costs. 

Lo que dice a grandes rasgos es que la metodología en cascada, aunque es un concepto simple,
es un proceso arriesgado y propenso a fallos y que puede llevar a un rediseño completo y por supuesto a sobrecostes si hay
que rehacer el trabajo. 

Casi al final del mismo artículo, curiosamente, presenta este gráfico:

![Metodología en espiral](/blog/articles/opinion/tendencias-programacion/metodologia-en-espiral.jpg "Figura de metodología en espiral")

que realmente presenta las bases de la metodología en espiral. Además, el mismo artículo habla de la importancia de las pruebas
y le dedica un paso, el último, a explicar por qué se debe involucrar al cliente en el desarrollo. Una joya.

Por terminar con la presentación, recuerdo a mi profesor diciéndonos: *"No se debería utilizar esta metodología, 
por supuesto hay métodos más eficientes, se explica porque es fácil de entender pero nadie debería aplicarlo, es 
como cuando explicamos los átomos con un gráfico similar al sistema solar. Sabemos que los átomos no son así pero 
es fácil de enseñar y comprender."*

El caso es que lo aplicamos. Durante un montón de años. Teníamos todo un conjunto de protocolos asociados a la metodolgía en cascada
o similar como Métrica. Así nos fué.

Si la persona que describe en primer lugar la metodología en cascada nos dice que no la utilicemos porque es peligrosa ¿por qué 
nos empeñamos en hacerlo? Muy simple: porque olvidamos nuestra historia.

Y nos lleva a las tendencias de programación actual, hemos pasado de la programación basada en estudio a la programación basada en
foros y últimamente a la programación basada en gurús y modas.

Un ejemplo, llevo años enseñando a mis compañeros que el código no se debe repetir, que la programación de copiar y pegar no es buena,
que dos, si hablamos de código, es un número excesivamente grande. Por una razón muy sencilla e histórica, el software es un modelo
computable de un dominio de la realidad, si tenemos dos modelos, tenemos dos representaciones de la misma realidad.

Dicho de una forma más llana, si tu código se repite, cabe la posibilidad que si hay un error, corrijas sólo una parte y que el error
perdure. Te costará tiempo y dinero corregirlo. La vida es dura, en resumen.

El caso, es que la tendencia actual va en contra de la filosofía de programación que considera anticuada y llegamos a un punto en que
*"si es necesario repetir el código lo repito porque la primera vez estoy en una fase de descubrimiento, la segunda la puedo
entender como una fase de refinamiento y a la tercera he comprendido realmente la concepción holística del modelo y tengo
más argumentos para refactorizar mi código y evitar las duplicidades"*.

Puede parecerlo pero no estoy de broma, el discurso es más o menos así. Sé que no tiene nada que ver, pero ese tipo de argumento
me recuerdan a lo que le decía Doña Jimena a Don Alfonso en la obra de teatro Anillos para una dama de Antonio Gala (años
llevo queriendo meter este párrafo en un artículo sobre desarrollo):

> ¡Déjame a mí de patrias! ¿No ves que estoy de
> vuelta de las grandes palabras? Las he mamado, Alfonso. Me he criado con ellas. He
> jugado con ellas, de niña, a la pelota... Apenas si he tenido marido, el que me diste,
> porque ya me lo diste con las grandes palabras. Y hubiera sido cariñoso y amante, Pero,
> ¡ah!, no pudo ser: allí estaban, en medio siempre de los dos, esas grandes palabras... ¿Y
> mi hijo? Mi hijo se quedó muerto, solo en mitad de un campo, con las grandes palabras
> por almohada... Estoy segura de que al morirse dijo «madre» y no «patria».

Me explico, no me hables de la concepción holística del software, me cansan las grandes palabras: dime que eres un vago, que no
has pensado en el problema, que estabas escribiendo con el piloto automático y no te has dado cuenta, no me digas 
sin sonrojarte que lo has hecho intencionadamente. Estás dejando minas en tu código y te van a explotar en la cara. 
Pero sobre todo, no me pidas que lo haga. No compro.

Y eso nos lleva a cosas como la programación con componentes. Algo fundamental en el desarrollo desde el principio de los tiempos pero
que últimamente llevamos al extremo de antes de pensar cómo lo podríamos hacer vamos a buscar si en Internet hay algo parecido
que puede servirnos. Y lo encontramos. No estamos muy seguros de si sus requisitos son exactamente los nuestros, si funcionará
en nuestro sistema, pero ahí está. Hemos ahorrado un tiempo valioso en el desarrollo y vamos a utilizar posiblemente el
cinco por ciento de un componente que no estamos seguros de si se va a mantener o no, que tendremos suerte si tiene algo
de documentación y que habrá que tener cuidado que no cambie sus especificaciones (porque recordemos que es su código, no el nuestro).

Después nos extrañamos cuando 
[once líneas de código JavaScript](https://www.lavanguardia.com/tecnologia/internet/20160325/40672383068/codigo-kik-internet-left-pad-programadores.html)
para hacer un left pad nos dejan sin Netflix y Spotify o cuando tenemos programadores a los que no les podemos pedir aplicaciones 
complejas porque tenemos buscadores de componentes no desarrolladores.

Nos lo hemos buscado.

Porque eso sí, vamos a tener largas discusiones sobre el nombre que le ponemos a las variables y sobre la carga cognitiva del
código (que sí, que se dice así) y vamos a utilizar sistemas molones de propagación de mensajes en lugar de llamadas simples 
a métodos (que ya no se lleva) que sabemos que consume más recursos pero la moda vende y sistemas de ORM que nos 
abstraigan de la base de datos aunque las consultas sean mas ineficientes y utilizaremos lenguajes de programación diseñados
en quince días para hacer procesamiento en servidor porque en realidad no hemos aprendido nada.

¿Parezco enfadado? Disculpad. No lo estoy. Por si no os habíais dado cuenta este es un artículo en modo irónico, realmente
me encanta hablar de los aspectos holísticos de la programación. Y de Dijkstra, y de Turing, y de Shannon, pero esa es otra historia.