+++
title = "Programando un intérprete de cron"
date = "2019-10-11"
description = "Ejemplo de programación de un intérprete de cron y por qué alguien pierde el tiempo en programar un intérprete de cron"
thumbnail = "/articles/opinion/interprete-de-cron/interprete-de-cron.jpg"
tags = [ "Programación" ]
+++

Para quien no lo conozca, **cron** es un comando de Unix que ejecuta aplicaciones cada cierto tiempo.

Lo curioso es el formato utilizado para definir los intervalos de tiempo, es decir, el momento en que tiene
que lanzar cada aplicación o comando.
	
Para definirlo usa una cadena con cinco o seis partes. La primera identifica cada cuantos segundos, la segunda
los minutos, la tercera las horas, la cuarta los días del mes, la quinta los meses y la sexta el día
de la semana. A esto se le llama una [cadena cron](https://en.wikipedia.org/wiki/Cron)
o una expresión cron. 

El fin de semana pasado, tenía unas horas sueltas y me puse a escribir un intérprete de expresiones cron. Lo podéis localizar en mi GitHub en
el repositorio [CronParser](https://github.com/jbautistam/CronParser). Por supuesto no
es todo lo completo que debería y no incluye la ejecución, sólo la interpretación. Tomadlo más como una prueba de concepto
que como una librería de trabajo real, de hecho no tiene *readme* por algo.
	
**Nota:** Puedes ver un gráfico de la expresión en la cabecera de este artículo y si te saltas la siguiente sección
puedes leer más información sobre las expresiones cron.

## ¿A qué viene todo esto?

Después seguiré hablando de cron, pero en realidad, en este artículo, el intérprete de cron no es importante. 

Lo importante de este artículo es por qué escribir un intérprete de cron en primer lugar cuando es mucho más fácil ir a Nuget 
o GitHub y descargar el más votado. 
	
Y la historia empezó hace cuatro meses cuando cambié de trabajo. He comenzado mi andadura con un equipo nuevo de personas muy
cualificadas (que posiblemente lean esto así que no voy a pelotear demasiado) y estamos en el momento de conocernos y de saber
cómo trabaja cada uno.
	
Esto me ha obligado por ejemplo a abandonar ciertas rutinas de desarrollo y adquirir otras. Por ejemplo, he abandonado la notación
húngara (vengo de C++, se me tenía que notar en algo), he dejado mi forma de indentar por comentarios que al resto del equipo
le parecía raro (e incluso aberrante para alguno) y he comenzado a aplicar otros métodos de desarrollo. 
	
Sin acritud, hasta ahora mantenía técnicas en desuso porque era el 'programador de referencia' y podía / debía imponer mi 
criterio al equipo, ahora tengo que adaptarme a la forma de trabajar de un equipo: es lo que hay. 
	
Ha supuesto un esfuerzo, no demasiado, pero un esfuerzo al fin y al cabo y lo he hecho con gusto. Eso sí, ha habido cosas a las que no
he renunciado aún, como a comentarlo absolutamente todo, pero ahí se han impuesto mis argumentos o el cansancio del resto del equipo
según a quien preguntes.
	
En esta andadura, hay cosas que aún nos sorprenden entre nosotros y como decía seguimos aprendiendo de la forma de trabajar de cada
uno. Apasionante. Y agotador. Muchas discusiones, ceder en algunos casos y admitir que estaba haciendo las cosas mal en otros.
También he tenido mis victorias por cierto, y sentirme inseguro con mi trabajo es una sensación nueva que me está abriendo otros
horizontes y aprender, algo que echaba terriblemente de menos los últimos años.
	
Y de una de esas discusiones vino este intérprete de cron. 

Una de las nuevas aplicaciones que me ha tocado en suerte, tiene precisamente que ejecutarse cada cierto tiempo y me sugirieron
utilizar cadenas cron. Yo no las conocía, tengo el ligero recuerdo de haberlas estudiado en mis tiempos de trabajar con Unix
en la universidad pero creo que lo he enterrado bajo otras materias.
	
Mi primera respuesta fue: *tardaré unas horas en programarlo* y eso comenzó otra de nuestras discusiones, creo que esta vez sin
sala reservada... Lo que más me sorprende de mi nuevo equipo es cuánto le gusta discutir.
	
El argumento general, era *¿cómo vas a perder el tiempo en hacer eso si ya hay librerías para ello?* y un *¿cómo vas a ser capaz
de hacer eso?* que sinceramente me escoció un poco.
	
Así que me sometí a la presión social y a la lógica y busqué una librería para ello:
[NCronTab](https://github.com/atifaziz/NCrontab), sinceramente mucho mejor que la mía aunque no
incluya la interpretación de años, pero me dejó esa sensación agridulce que me asalta cuando no puedo programar y tengo que utilizar 
librerías de otros, así que aproveché cuatro horas del sábado para desarrollar mi propio intérprete y así nació esta librería.
	
Hoy ha surgido de nuevo el tema, hablando de otras cosas comenté *este sábado he dedicado cuatro horas a escribir un
intérprete de cron* que llevó a reacciones que iban del *¿para qué has hecho eso?* al *no me creo que hayas sido capaz* y de
ahí sale este artículo.
	
Para los que no me conocen, dedico gran parte de mi tiempo libre (incluyendo un sinnúmero de madrugadas) a programar para mí.
Hay a quien le gusta la fotografía, quien dedica su ocio a cocinar o a hacer macramé. Yo programo. También... es lo que hay. 
	
Programar además es mi trabajo, pero que tenga la suerte de trabajar en lo que me gusta y que mi trabajo me 
lleve por caminos interesantes no quiere decir que cubra todos los campos del desarrollo, así que mi tiempo libre lo ocupo en aplicaciones 
que se me ocurren pero no tienen cabida en el día a día. Aplicaciones, librerías o implementaciones de algoritmos que me resultan interesantes. 
	
La mayor parte de estas aplicaciones o librerías no llegan a ver la luz para el público aunque algunas como
[BauPlugStudio](http://bauplugstudio.webs-interesantes.es) las utilizo a diario (entre otras
cosas para escribir este artículo). Algunas las comparto por GitHub o se merecen un artículo en esta web pero no pasan de ahí. 
	
De hecho, muchas de las aplicaciones que programo se quedan a medias, simplemente llego al punto de *a partir de aquí todo
es fácil* y las abandono por otros proyectos que me llaman la atención. Lo que me motiva no es solo conseguir la
mejor aplicación posible, mi motivación principal es aprender cómo llegar a la solución. Si además me divierte ¿por qué
no debería hacerlo?
	
Estos años, por ejemplo, aparte de **BauPlugStudio** que ha ocupado gran parte de mi tiempo, 
escribí una [librería de FTP](Código fuente\Librería FTP en C 1), una [librería POP3](Código fuente\Librería POP3 en CSharp), 
un [documentador de código](Aplicaciones\NSharpDoc), 
un [editor / visualizador de presentaciones](Aplicaciones\BauMotionComics),
una implementación del [protocolo XMPP / Jabber](Código fuente\BauMessenger) (en ese caso la librería
de comunicaciones la descargué de GitHub porque hace unos diez años escribí mi propia implementación, la perdí y no me
apetecía empezar de cero), un [servidor SMPTP para desarrollo](Código fuente\Servidor SMTP) 
o un [motor para juegos 2D](Código fuente\CrioGame) utilizando MonoGame y Win2D. 
	
Posiblemente en lugar de a estas cosas, podría apuntarme a Hackatons o hacer puzzles de 
[HackerRank](https://www.hackerrank.com/) (que alguna vez me ha dado por ahí) pero creo que desarrollar 
puzzles sólo te concede la habilidad para resolver puzzles no la capacidad para llevar un proyecto de principio a fin. 
	
La pregunta es ¿eso me hace mejor programador?

Sinceramente, creo que no. Me hace más versatil, me permite llevar a mi trabajo diario conocimientos que adquiero en casa (por ejemplo
las librerías de FTP y POP3 llevan utilizándose en producción casi siete años), me hace perderle el miedo a emprender
desarrollos que no he hecho antes porque posiblemente sí haya hecho antes algo parecido y me hace ver los problemas en conjunto
con una perspectiva diferente.
	
Me da también una falsa sensación de seguridad. Un 'yo podría hacerlo mejor' que resulta peligroso en entornos de trabajo y tengo que
contener. Hay veces en las que simplemente tienes que tener en cuenta que no te pagan por hacer librerías de cron si no por hacer
aplicaciones de negocio y esto implica ir a Nuget y ahorrar tiempo. La diversión queda para el tiempo libre en esos casos.
	
Y tiene otro inconveniente: el hacer lo que quieres cuando quieres hace que te enfoques en lo que te gusta y dejes de lado otras
cosas que para tu trabajo pueden ser valiosas. Por ejemplo, me disgusta la aplicación frontend y eso hace que no le dedique tanto tiempo
como debería a cosas como ASP.Net. Tampoco utilizo ORM, realmente jamás me ha hecho falta y he desarrollado mis
soluciones para ello, pero no he aprendido a utilizar Entity Framework y eso es un problema. Esto levantará ampollas y llevará a
otro artículo. Prometido.
	
Tampoco he sido capaz de aprenderme los nombres de los patrones de diseño. Posiblemente los utilice correctamente todos pero si me 
hablas de un **Strategy** obtendrás una mirada perpleja.
	
También me convierte en alguien más desordenado: puedo empezar una aplicación literalmente por cinco partes diferentes
e ir desarrollando todas a la vez. A más de uno de mis compañeros ésto le hace odiarme con toda la razón. Prometo que me
lo estoy mirando.
	
Y este desorden también se aplica a mis estudios. Este año he leído sobre Big Data, Hadoop y Spark, R, Bussiness Intelligence, patrones
de diseño, arquitecturas de código, contenedores, microservicios, desarrollo de videojuegos, técnicas SEO, diseño Web,
Angular (lo siento, eso lo abandoné), Python, Go, desarrollo móvil, técnicas de márketing y negociación, optimización de SQL,
técnicas de seguridad y pentesting... Si lo unes todo, obtendrás un cacao de sé mucho de todo pero posiblemente nada de casi
todo preocupante cuanto menos.
	
Dicho esto, se me siguen acumulando los proyectos personales y desde aquí amenazo con nuevos desarrollos: 

* Tengo pendiente un intérprete de [PGN](https://es.wikipedia.org/wiki/Notaci%C3%B3n_portable_de_juego) que posiblemente lleve también
una base de datos de juegos de ajedrez que aprovecharé para recuperar el desarrollo UWP.

* Mi [lector de cómics](Aplicaciones\BauComicBooks) se ha quedado más que anticuado, le voy a dar una vuelta y aprovecharlo
para aprender Xamarin Forms ahora que ya puede utilizar librerías estándar (la librería del descompresor de RAR que utilizaba sólo funcionaba
a partir del Framework 4.0 y una de las pocas cosas a las que aún no me atrevo es a escribir mi propio compresor).

* En una charla de [Alvaro Videla](http://wedevelopers.com/2014/06/23/we-developers-034-rabbitmq) sobre
RabbitMQ hablaba de un [simulador de RabbitMQ](https://vimeo.com/56986242) que utilizaba en sus presentaciones
para no tener que 'mover las manos' cuando explicaba cómo se comunicaban los mensajes en diferentes entornos (creo además que es de las 
pocas aplicaciones interesantes escritas utilizando [Processing](https://processing.org)). 
Ultimamente que voy a más conferencias veo mucho 'movimiento de manos' para explicar conceptos y me parece que podría aprovechar 
lo aprendido en [BauMotionComics](Aplicaciones\BauMotionComics) para crear un lenguaje que permita hacer ese tipo 
de presentaciones rápidamente. Algo así como mezclar [yUML](https://yuml.me) con PowerPoint, puede
ser un éxito o una mala idea pero me llama la atención.

* Hablando de charlas: dado que hay conferencias de desarrollo todos los días, estoy programando una aplicación para categorizar / ver las
conferencias de desarrollo que nos perdemos cada día. Espero tener noticias pronto si no se le cruza otro proyecto antes.

* Sigo pensando en **Colmena**: un framework para comunicación entre sistemas / usuarios que me lleva rondando desde que comencé a leer sobre
protocolos abiertos de redes sociales pero que aún no acaba de cuajar en mi mente.

* Mi servidor SMTP ha ido derivando en un sistema de marketing por correo electrónico que incluye la lectura y distribución de correos entrantes
utilizando reglas. Parece complicado pero en realidad es bastante sencillo. La duda es si llegará a ser útil y en ese impasse lleva un par de meses
en el disco duro.
		
Es decir, no tengo planes de dejar de programar ni estudiar ahora que tengo menos tiempo libre, simplemente tendré que aprender
a priorizar.

Una nota totalmente aparte para los amantes del código Nuget, npm y GitHub: recordad que esto conlleva riesgos:

* [Protege tu cuenta de GitHub](http://www.elladodelmal.com/2017/05/developer-protege-tu-cuenta-de-github.html).
* [El javaScript que llevó el caos a Internet](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos).
* [Módulos maliciosos en npm](https://blog.liftsecurity.io/2015/01/27/a-malicious-module-on-npm/).

## Y ahora hablemos de cron

Pero bueno, un artículo sobre **Programando un intérprete de cron** se merece al menos un poco de teoría sobre cron.

En realidad el estándar habla de seis partes, aunque en algunas implementaciones los segundos 
son opcionales: si encuentra cinco partes considera que se debe ejecutar en cualquier segundo. Además, existen formatos 
especiales como el de [Quartz](https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm), 
por ejemplo, que utiliza cadenas de siete partes en las que la séptima es opcional y representa los años de ejecución.
	
La gracia, por supuesto, está en su versatilidad. Cada una de las posiciones, aparte de un número fijo puede indicar diferentes
caracteres especiales, intervalos o rangos de valores. Así:

* Un número indica en ese momento en concreto. Por ejemplo, un 3 en la sección de día del mes indica que se debe
ejecutar cada día tres.
* Un asterisco indica en cualquier momento. Por ejemplo, un asterisco en la posición de los meses indica que se debe
ejecutar cada mes.
* Una serie de números separados por comas indica que se tiene que ejecutar en esos instantes. Por ejemplo, *1,3,5*
en los días de la semana indica que se debe ejecutar los lunes, miércoles y sábado (los días de la semana van de cero
a seis, el domingo es el cero).
* Dos números separados por un guión indican un intervalo. Por ejemplo *3-5* en los meses indica que se debe ejecutar
en marzo, abril y mayo.
* Dos números separados por una barra, indican una sucesión. Por ejemplo *3/2* en los meses indica que se debe ejecutar
cada dos meses a partir de marzo (marzo, mayo, julio...)
* Además, los días de la semana o los meses se pueden definir tanto con una abreviatura de tres letras (MON para lunes o JAN 
para Enero) como por el número correspondiente.
		
**Nota:** Existen otros caracteres especiales como la interrogación. Podéis ver el formato completo en
la [Wikipedia](https://en.wikipedia.org/wiki/Cron).
	
Así por ejemplo:

* `* * * * * * `: indica que se tiene que ejecutar cada segundo.
* `* 30 * * * * `: indica que se tiene que ejecutar en el segundo treinta de todos los minutos.
* `* 0 5 12 * * *`: indica que se tiene que ejecutar todos los días a las 12:05:00.
* `* 0 5 12 * * MON-WED`: indica que se tiene que ejecutar de lunes a míercoles a las 12:05:00.
* `* 0 5 12/5 * * MON-WED`: indica que se tiene que ejecutar de lunes a míercoles a las 12:05:00, 14:05:00, 16:05:00 etc...
	
Para no eternizarnos, si queréis ver ejemplos siempre podéis recurrir a Internet. Hay ciertas páginas como
[CronMaker](http://www.cronmaker.com/) que generan cadenas cron a partir de una interface gráfica.

## Intérprete de cadenas cron

Una vez zanjado el tema del porqué todo esto, vayamos al tema de cómo lo implementé. Os recuerdo que no es una implementación
completa, simplemente una prueba de concepto de cómo se puede solucionar.
	
En realidad el intérprete de cadenas es bastante sencillo. Las expresiones cron son cadenas separadas por espacios con un número
limitado de secciones. Así que el intérprete lo único que tiene que hacer realmente es separar la cadena en partes y 
generar cada una de ellas. En mi caso utilicé el formato de seis o siete partes: incluye segundos y los años son opcionales.

La librería separa el intérprete en dos clases **CronSentence** que dirige la interpretación y que tiene un array de
siete objetos de tipo **CronPart**.
	
La clase **CronPart** por su parte es la que interpreta cada una de las secciones y mantiene el intervalo posible (los segundos y
minutos van de 0 a 59, las horas de 0 a 23, etc) y un **BitArray** con los valores que se ha encontrado en la interpretación.
Es decir, si la cadena de minutos era *5,9*, el **BitArray** tiene 60 elementos, todos ellos a false excepto los 
correspondientes a los índices 5 y al 9.
	
Por tanto, para interpretar:

* Si encontramos un asterisco, marcamos a true todos los valores posibles.
* Si encontramos una cadena con únicamente un entero, marcamos a true únicamente ese valor. 
* Si encontramos un guión en la cadena, marcamos a true los valores enteros que van desde el valor de la izquierda del 
guión hasta el valor entero a la derecha del guión.
* Si encontramos una coma, separamos la cadena por comas e interpretamos cada una de las cadenas encontradas (porque
nos podemos encontrar construcciones como *1,3-7,9*).
* Al interpretar el valor numérico debemos tener en cuenta que puede aparecer una barra (*3/7*) que indica que
comienza en 3 y se debe repetir cada 7 elementos, así que marcamos desde el tres incrementando en siete hasta que
encontremos el límite superior.
* Hay varios caracteres especiales de los que no hemos hablado pero también se interpretan:
	* La almohadilla indica un elemento en particular en la sucesión, por ejemplo, `MON\3` indica que se debe ejecutar 
	el tercer lunes del mes. Este caso es especial y lo que hacemos es almacenarlo en dos variables intermedias. Por
	cierto, hay ciertas implementaciones que indican que `MON\5` es un error, uno de los problemas de no programarlo
	nosotros y confiar en los demás (vale, eso lleva a un Pull Request pero era por meter el dedo en la llaga).
	* El carácter L detrás de un valor entero, indica el último valor. Por ejemplo *0L* en los días de la semana
	indica el último domingo del mes. Este carácter sólo es válido para días de la semana.
	
Por supuesto, cuando en la lista anterior hablo del 'valor entero' me refiero o bien al dígito de la cadena o bien al índice
de la abreviatura en el caso de días de la semana o mes.
	
Cómo vemos, la librería sólo trata la interpretación, no la ejecución. Es decir, nos interpreta la cadena pero no nos dice
cuál debe ser la siguiente vez que se ejecute el comando atendiendo a la cadena.
	
**Nota:** en realidad sí que lo hace, pero es bastante sucio: a partir de una fecha, la va incrementando
en un segundo y comprobando si en ese segundo se debe ejecutar o no atendiendo a la cadena cron. Para cadenas 
como `* * * * * FEB *` si estamos en diciembre va a hacer un bucle que puede llevar un tiempo considerable. Lo lógico es que 
nos diera la siguiente fecha de una forma más inteligente, simplemente incrementando cada una de las partes. La siguiente versión.

## Y para terminar

Antes de ponerme a programar después de tres horas de escritura incluyendo apagón, una pregunta:

¿Vosotros cómo abordáis estos temas? ¿sois de hackaton, puzzles o aplicaciones / librerías?