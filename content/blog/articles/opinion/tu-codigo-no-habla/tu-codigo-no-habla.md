+++
title = "Tu código no habla (o al menos a mí no me dice nada)"
date = "2019-10-11"
description = "Por mucho que nos disguste, los comentarios en el código continuan siendo necesarios"
thumbnail = "/articles/opinion/tu-codigo-no-habla/tu-codigo-no-habla.jpg"
tags = [ "Opinión" ]
+++

No sé si alguna vez os habéis encontrado este código:

![Código fuente](/blog/articles/opinion/tu-codigo-no-habla/codigo-fuente.jpg "Código fuente")
			
Para los más curiosos, el código pertenece a una librería escrita en Go para la 
[lectura de archivos de Excel](https://github.com/tealeg/xlsx)
y en resumen sirve para pasar una fecha del calendario gregoriano a un número 
entero. Ese número entero, llamado número Juliano, se puede utilizar por ejemplo para hacer comparaciones entre fechas.
	
El código original estaba escrito en Fortran y lo publicaron Henry F. Fliegel y Thomas C. Van Flander en 1.968 
en una carta al director en la revista Communications of ACM (CACM, volumen 11, número 10, Octubre de 1.968, 
página 657). Os dejo un enlace por si queréis leer el 
[artículo original](http://www.cs.otago.ac.nz/cosc345/resources/Fliegel.pdf). 
	
Lo interesante de este código y por lo que encabeza el artículo, es precisamente por el comentario:

![Comentario de codigo fuente](/blog/articles/opinion/tu-codigo-no-habla/comentario-codigo-fuente.jpg "Comentario de codigo fuente")
 
Al parecer, este código no está únicamente en ese repositorio, también se puede encontrar en las fuentes de 
Linux, Android, Excel… y es fundamental para todas las operaciones con fechas. Sin embargo, el artículo no 
detalla su funcionamiento y como se puede apreciar, los nombres de las variables y las constantes no ayudan mucho.
	
Así que lo copiamos y pegamos siempre que lo necesitamos pero nadie parece entenderlo.

**Nota:** por cierto, no es que me pase el día leyendo código de GtHub, 
la imagen viene de un mensaje en Twitter de Kenn White ([@kennwhite](https://twitter.com/kennwhite/status/721035633657909248)).
Tampoco me paso el día en Twitter aunque algunos lo creáis.
	
Particularmente me recuerda mucho al teorema de Fermat (xn + yn = zn). Al parecer, en el margen del 
cuaderno donde Fermat escribió su conjetura dejó este párrafo:
	
> He descubierto una demostración realmente maravillosa de esto que este margen es demasiado estrecho para contener
	
El teorema se publicó en 1.658, los matemáticos han tardado 358 años en demostrarlo. Todo un tipo el tal Fermat.

Pero no nos vayamos por las ramas, este artículo no va de Fermat ni de números julianos, habla de documentación 
del código. De la falta de comentarios en realidad.

No estoy seguro de cómo empezó el cambio de paradigma que pasó de "tienes que comentar tu código" a "si quieres ser
cool y agile no dejes comentarios en el código". Y nos encontramos referencias por ejemplo en el libro *Clean code* de
Robert Martin que a tanta gente cambió la vida:

> As useful as javadocs are for public APIs, they are anathema to code that is not intended for public consumption.

Me guste o no me guste lo que has escrito ¿podrías explicarme por qué? Si haces una aseveración de ese estilo, con esa rotundidad,
o me das una demostración o continúas con "con un par". A tu criterio.

En primer lugar, deberíamos debatir sobre qué es público cuando hablamos de código fuente. Si trabajo para una empresa
que se dedica a escribir componentes parece claro que "público" significa "aquellos métodos que se exporten". Si
trabajo desarrollando código en equipo en cualquier otro tipo de empresa "público" significa "cualquier parte del código" 
porque antes o después, no nos engañemos, vamos a acabar leyéndolo.
	
Al fin y al cabo nuestro querido Martin no escribió sólo ese párrafo. Desarrollaba un poco más la idea. Tenía todo un libro para
ello de hecho: si conseguimos que los nombres de nuestras clases, propiedades, métodos y variables 
sean más expresivos y cuenten qué es lo que hace cada parte del código, entonces los comentarios son irrelevantes.
Además, si escribimos pruebas sobre nuestro código podemos conseguir que las propias pruebas constituyan la documentación
del código. Recordemos que los comentarios no se compilan, no se puede asegurar su integridad: puede que tengamos un
comentario que no refleje la realidad del código. Es realmente el código quien dice la verdad, no los comentarios.
	
Parece lógico. Y últimamente cada vez es más habitual que escuche cosas como éstas:

> No escribimos documentación porque somos frescos y ágiles y creemos que el código en funcionamiento es mejor que 
> una documentación exhaustiva
	
Así que a partir de ahora podemos ya pasar sin miedo del "no documento porque no me da la gana" como hacíamos hasta ahora a 
obviar la documentación sin miedo porque somos fresquitos y ágiles y a partir de este momento nuestro código ya
es mágicamente expresivo y tenemos un 100% de cobertura de código. Me gustaría saber en qué punto del manifiesto ágil
pone que no hay que documentar, pero eso es otra historia y cómo empecemos a hablar de Agile este artículo no acabará nunca.
	
No sé vosotros, pero si alguien me da un código y me dice que busque la documentación entre las pruebas unitarias me enfado
bastante. Imaginad por un momento que el código tiene una cobertura del 80%. Muy alto, pero puestos a imaginar. Imagináos
que estoy en un método privado al que llama un método privado de una clase interna al que llaman varios métodos privados
de otras clases internas a los que llama un método público de una clase pública que, maravilla de las maravillas, tiene varios tests.
	
¿Realmente la respuesta a "no escribo comentarios" es que ejecute las pruebas de esos métodos línea por línea y adivine
qué es lo que se pretendía hacer? ¿Me estáis tomando el pelo?
	
El truco entonces parece ser escribir código expresivo. En un código expresivo el propio código describe
lo que tiene que hacer. Dado que el lenguaje natural es ambiguo (recordemos el problema del arzobispo), creamos 
lenguajes de programación con palabras clave más o menos sencillas. Lo único que nos queda entonces para identificar 
la intención del código son los nombres de clases, propiedades, variables y métodos. Estos nombres deberían reflejar 
lo que el programador pretendía a la hora de escribir el código y su forma de pensar para resolver el problema. 
Lo malo es que tu forma de pensar y la mía son totalmente diferentes, así que abrimos una discusión para ver cuál 
es el nombre de variable más adecuado.
	
Por ejemplo, vamos a ver cómo indicamos en nuestro código que se debe aplicar un descuento cuando el volumen de la venta
	supera la media de ventas de una tienda:
	
```csharp
	if (purchasedQuantity > salesAverage)
	    ApplyPromotion();
```

Parece claro pero aún así, `purchasedQuantity` puede que no sea el nombre más adecuado: cantidad comprada. Quizá fuese mejor
que indicásemos que es la cantidad adquirida por el cliente, además, `salesAverage` ¿qué es? ¿la media de ventas de
la tienda o la de la empresa? Quizás fuera mejor algo así:
	
```csharp
	if (purchasedCustomerQuantity > salesStoreAverage)
	    ApplyPromotion();
```

Qué queréis que os diga, para un artículo en un blog o para un ejemplo en un libro, no está mal, pero si cualquiera de
vosotros me contáis que mientras escribís código, en ese momento en que todo fluye y parecéis uno con el teclado, empleais
más de una décima de segundo en elegir el nombre de una variable os responderé que me estáis mintiendo descaradamente.
	
Pero sigamos con el ejemplo, esa condición es sencilla, pero si las cosas se complican puede que deseemos sacar esa condición a un 
método que nos hará todo mucho más expresivo:
	
```csharp
	if (MustApplyPromotionToCustomerAtStore(purchasedCustomerQuantity, salesStoreAverage))
	    ApplyPromotion();
```

Mucho mejor, dónde va a parar. Ahora `MustApplyPromotionToCustomerAtStore` nos indica si debemos aplicar la promoción.
Basta con leer `MustApplyPromotionToCustomerAtStore` y evitamos hacer la comparación en nuestra cabeza para
saber si la línea está bien. Nos falta escribir el código de `MustApplyPromotionToCustomerAtStore` pero es sencillo y podemos utilizar
en otras partes del código llamadas a `MustApplyPromotionToCustomerAtStore`. `MustApplyPromotionToCustomerAtStore` es la solución. 
`MustApplyPromotionToCustomerAtStore` al poder. Nada como `MustApplyPromotionToCustomerAtStore` desde la invención de la cerveza light.
	
¿Cuántos habéis notado el clik en vuestro cerebro al leer `MustApplyPromotionToCustomerAtStore`? ¿Cuántos os
habéis detenido un segundo mientras leíais? ¿Cuántos os habéis saltado la dichosa palabreja cada vez que aparecía en 
el párrafo anterior? Y es que los nombres de métodos complejos no son precisamente fáciles de leer, al cabo de un rato 
nos negamos a distinguir entre `MustApplyPromotionToCustomerAtStore` y `MustApplyDisscountToCustomerAtStore`.
	
Por eso hay ocasiones en las cuales lo más simple es un comentario:
	
```csharp
	// Si debemos aplicar la promoción al cliente
	if (MustApplyPromotionToCustomerAtStore(purchasedCustomerQuantity, salesStoreAverage))
	    ApplyPromotion();
```

Mi código está lleno de comentarios así. Es mi manera de escribir. Así cuando leo el código voy pasando directamente por
los comentarios sin detenerme realmente en las instrucciones que no me interesan. En realidad, leo el pseudocódigo,
no el código. 
	
**Nota:** Creía que era cosa mía pero existe incluso un estándar llamado
[Commenting Showing Intent - CSI](https://standards.mousepawmedia.com/csi.html) que llega prácticamente a
las mismas conclusiones a las que llegué cuando comencé a escribir comentarios de este tipo.
	
Os escucho decir "pero entonces tendrás que cambiar el comentario cuando cambies el código". Por supuesto. Y el nombre 
de las variables. Y el nombre del método `MustApplyPromotionToCustomerAtStore` por mucho cariño que le haya cogido. 
Y posiblemente también el documento de requisitos y el manual de usuario. ¿Creíais que la vida del programador era 
fácil? Que no se compile no quiere decir que no sea nuestra responsabilidad.

Porque todos sabemos que hacer código expresivo no es fácil. Un ejemplo de una interface sencilla:

```csharp
public interface IJobProcessor
{
	int Process();
}
```

Simple, un sólo método *Process* que como su propio nombre indica suponemos que procesa y dado
que está dentro de una interface llamada `IJobProcessor` supongo también que procesa un trabajo.
	
Me preocupa un poco el valor de retorno: ¿tengo que devolver un código de error? ¿el número de registros procesados?
¿el máximo de grados Kelvin que ha alcanzado el procesador durante la ejecución? Al fin y al cabo, si me
das una interface no documentada puedo hacer lo que me venga en gana.
	
No me vendría mal algo de ayuda aunque supongo que siempre puedo mirar el código de las pruebas porque: 
vosotros también escribís pruebas para los interfaces ¿no es así? Vamos, como todo el mundo.
	
Alguno me dirá que realmente no es un código expresivo: en lugar de un entero debería devolver una clase con
propiedades diferentes una para el código de error, otra para el número de registros y otra para los grados
Kelvin y que además el interface se debería llamar 
`IJobDataBaseQueryProcessor`. Posiblemente yo me conformaría con esto:

```csharp
// Interface para las clases que ejecutan consultas sobre la base de datos
public interface IJobDataBaseQueryProcessor
{
	// Ejecuta la consulta. Devuelve el número de registros afectados
	int Process();
}
```

Por supuesto, preferiría que fueran comentarios de XML que pudiera añadir automáticamente a mi documentación pero
tampoco vamos a ponernos exquisitos.
	
Ya puestos, lo que realmente me gustaría saber es:

* ¿Cuáles son las precondiciones del código? ¿que argumentos necesita?
* ¿Qué vas a hacer con el valor de retorno?
* ¿El código se va a ejecutar dentro de un hilo, hay alguna consideración sobre inmutabilidad, invariantes
o dependencias?
* ¿Qué tipo de excepciones debo lanzar? ¿Las excepciones se deben controlar dentro del propio proceso?
	
No me digas que todo esto va dentro de los nombres porque no es así. Y si tengo que leer todo tu código para enterarme, sinceramente
no me interesa.
	
Y sólo un punto más para dejar claro mi punto de vista: el código dice lo que el código hace. No dice lo que el código debería
hacer. Lo que el código debería hacer lo dicen los requisitos e incluso el manual de usuario. Si me haces enfrentarme a 60.000 líneas
de código sin un sólo comentario al menos espero que tengas al día los requisitos.

A pesar de lo dicho hasta ahora, he decidido que quiero ser cool y agile y voy a dejar de comentar mi código así que os jod... 
esto... mi código a partir del siguiente parpadeo será explícito y lleno de test
que podéis consultar en cuanto tengáis alguna duda. Todo estará en el código. Lo que veis es lo que hay.
No preguntéis. Trabajo mucho, no esperaréis que me acuerde de todo.
	
Aunque lo que realmente espero es que cuando alguien os diga "soy cool y agile y no comento mi código" recordemos ésto:

> The difference between a tolerable programmer and a great programmer is not how many programming languages they know, 
> and it's not whether they prefer Python or Java. It's whether they can communicate their ideas. By persuading other 
> people, they get leverage. By writing clear comments and technical specs, they let other programmers understand 
> their code, which means other programmers can use and work with their code instead of rewriting it.
> Absent this, their code is worthless.
> **Joel Spolsky – Advice for Computer Science College Students**

---

**Nota:** por si alguien se lo preguntaba, el problema del arzobispo se utilizaba para ilustrar
la dificultad del procesamiento del lenguaje natural en sistemas de Inteligencia Artificial. Se basaba en una conversación
del Sombrerero Loco del libro Alicia en el País de las Maravillas (escribo de memoria):
	
- Y eso fue también lo que pensó el arzobispo
- ¿Eso? ¿Qué es eso?
- Eso es lo que pensó el arzobispo. ¿No sabes lo que es eso?
- Sé lo que es eso cuando se trata de una liebre o un pájaro pero no sé que es eso que pensó el arzobispo.