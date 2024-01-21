+++
title = "Seguridad por oscuridad"
date = "2019-10-11"
description = "Si alguna vez te has preguntado cuando es una buena idea utilizar técnicas de seguridad por oscuridad u ocultación, la respuesta es nunca. Jamás."
thumbnail = "/articles/opinion/seguridad-oscuridad/seguridad-oscuridad.jpg"
tags = [ "Opinión", "Seguridad" ]
+++

La seguridad por oscuridad u ocultación, es una metodología que intenta plantear la seguridad de un sistema a partir del secreto
de sus técnicas, utilizando procedimientos que no están documentados o se han desarrollado internamente. En otros casos, simplemente
se confía en que los delincuentes no encuentren nuestra información.
	
Y por si alguien lo pensaba, no es algo que yo me invente, tiene hasta definición en la 
[Wikipedia](https://es.wikipedia.org/wiki/Seguridad_por_oscuridad). 

Tiempo atrás, nos ocurría mucho con sistemas criptográficos. La propuesta era siempre igual: "vamos a inventar nuestro propio sistema de criptografía". 
La idea era que como el sistema utilizado para cifrar usaba algoritmos desconocidos nadie lo podría descifrar.
	
La idea es mala. Muy mala. Si algo tienen de bueno los sistemas de criptografía reconocidos, es que pasan por un proceso de auditoría constante, 
conocemos sus vulnerabilidades teóricas y prácticas y podemos actuar ante esas vulnerabilidades bien cambiando de sistema o 
bien aplicando parches y recomendaciones.

Un par de ejemplos: en julio de 2007, Richard Doherty decía sobre el sistema criptográfico anticopia de los discos BlueRay:

> BD+, unlike AACS which suffered a partial hack last year, won't likely be breached for 10 years.

Ocho meses después, el grupo Slysoft, anunció que podía eliminar la protección BD+ de cualquier disco. Según ellos, les costó ocho meses 
de trabajo de dos programadores a horas sueltas.
	
Otro caso: en 2011, GeoHot publicó la forma de acceder como root a la Playstation 3, con un mensaje mítico
[en su web](http://geohot.com/) (todo un dechado de técnica): "the keys open doors".
	
![GeoHot: root en Playstation 3](/blog/articles/opinion/seguridad-oscuridad/geohot.jpg "GeoHot: root en Playstation 3")
	
No busquéis el mensaje en su Web, Sony, tras una larga lucha, le obligó a quitarlo y a prometer que
no lo volvería a hacer más. La canción que le dedicó a Sony, también es mítica:

<iframe width="780" height="720" src="https://www.youtube.com/embed/9iUvuaChDEg" 
frameborder="0" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture"></iframe>

En realidad, las hazañas de GeoHot no comenzaron con la PS3, en 2007 fue el responsable del primer Jailbreak para iOS, famoso hasta hace muy poco
por no dar ninguna información sobre los sistemas de seguridad de su sistema operativo. De hecho, todo lo que sabíamos de la
seguridad del sistema era mediante técnicas de ingeniería inversa.

Volviendo a Sony y a la PS3: a raíz del juicio contra GeoHot comenzó a recibir una serie de ataques DoS y se filtraron
77 millones de cuentas de usuario con información sensible como los números de tarjetas de crédito. 
La respuesta de Sony, también es memorable:
	
![Sony: permanezcan vigilantes](/blog/articles/opinion/seguridad-oscuridad/sony.jpg "Sony: permanezcan vigilantes")
		
Sony de hecho, tiene un largo historial de fallos de seguridad por su afán de ocultar e inventar.
Por ejemplo, en 2005 [Mark Russinovich](https://blogs.technet.microsoft.com/markrussinovich/2005/11/14/sony-no-more-rootkit-for-now/)
les afeaba su afán de introducir rootkits en sus CD, reproduciendo declaraciones como ésta:
	
> Perhaps the biggest news in the story last week is Sony’s first public response since one
> of their executives stated in a National Public Radio interview, **"users don't know what a
> rootkit is, and therefore, don’t care."** 

Puede que os parezca extraño este tipo de afirmaciones, pero son más comunes de lo que pensaba. Por ejemplo, **Troy Hunt**
también tuvo alguna experiencia de este tipo con Fiddler:

<blockquote class="twitter-tweet" data-lang="es"><p lang="en" dir="ltr">I contacted the developer and explained what I&#39;d discovered 
and how I&#39;d found it. This was his response and I kid you not, this is word for word what he said:<br><br>"Don&#39;t worry, 
our users don&#39;t use Fiddler"<br><br>I don&#39;t work there any more.</p>&mdash; Troy Hunt (@troyhunt) 
<a href="https://twitter.com/troyhunt/status/1067510974821744640?ref_src=twsrc%5Etfw">27 de noviembre de 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Ultimamente parece que andemos algo escarmentados con la implementación de sistemas criptográficos, pero aparece otra modalidad de la seguridad
por oscuridad que podríamos denominar seguridad entre la multitud.
	
El concepto es algo así como "nadie me va a atacar porque hay muchos objetivos en Internet ¿quién se va a fijar en mí?" o 
"como mi sistema está en la nube, nadie encontrará las url de mis archivos entre la inmensidad de sitios web". Otra mala idea.
	
Y es una mala idea por algo que los programadores y los ciberdelincuentes hacemos muy a menudo: automatizar.

No os equivoquéis, no es necesario que os busquen directamente a vosotros. Un ciberdelincuente encontrará un fallo y lanzará un ataque
a ciegas, simplemente a ver qué encuentra. Y os encontrará.
	
Tenéis que considerar que una gran parte del tráfico que se origina en Internet es tráfico de ataque automatizado. De hecho,
podemos seguirlo en tiempo real con mapas como los de [Kaspersky](https://cybermap.kaspersky.com/es):	

![Mapa ciberataques en tiempo real](/blog/articles/opinion/seguridad-oscuridad/mapa-ataques.jpg "Mapa ciberataques en tiempo real")
	
¿Os habéis preguntado, por ejemplo, cuánto tiempo tarda un dispositivo IoT en caer en las redes de una botnet? 
[Rob Graham](https://www.digicert.com/blog/scaling-identity-for-the-internet-of-things)
hizo la prueba (spoiler: un minuto y cuarenta segundos):

![Una cámara tarda 98 segundo en caer en una botnet](/blog/articles/opinion/seguridad-oscuridad/rob-graham.jpg "Una cámara tarda 98 segundo en caer en una botnet")
 
¿Aún no me creeis? ¿Recordáis las []({ href = "Artículos\Seguridad\Hallazgos Shodan" } impresoras 3D y Shodan)? 

Hace unos meses, preparando una charla sobre seguridad, me pasé un rato en Shodan buscando servidores MongoDB. El primero que
encontré estaba completamente abierto y parece que no fui el único en tener la idea de conectarse:

![Shodan y MongoDb](/blog/articles/opinion/seguridad-oscuridad/shodan.jpg "Shodan y MongoDb")
 
Que viene a decir algo así como: "he encontrado tu base de datos, te la he robado porque me los has puesto en bandeja, 
si la quieres paga un rescate".

Puede parecer curioso que para dos veces que entro en Shodan por casualidad encuentro un dispositivo vulnerable y una base de datos secuestrada. Me
da por pensar que quizá no sea una casualidad.
	
Y no, no es una buena idea deducir que ninguno de vuestros clientes utiliza Shodan o 
[Scans.io](https://scans.io/) o no saben programar un script Python para buscar puertos abiertos porque
puede que sepan cómo buscar por Google esas contraseñas que os empeñáis en guardar en proyectos públicos de Trello:

![Contraseñas en Trello](/blog/articles/opinion/seguridad-oscuridad/trello.jpg "Trello")

Pero bueno, eso seguro que no os pasa porque vuestros archivos están en carpetas de AWS que aunque sean públicas tienen un nombre
que sólo conocen los miembros de nuestra empresa y no existe ninguna forma de automatizar estas cosas. No sé algo que se llame así como
[Bucket finder](https://github.com/FishermansEnemy/bucket_finder)
o [S3 Scanner](https://github.com/sa7mon/S3Scanner) y que podamos encontrar en GitHub, por ejemplo.
Por cierto, no son los únicos, los encontré en el artículo
[How to find unsecured S3 buckets: some useful tools](https://www.andreafortuna.org/cybersecurity/how-to-find-unsecured-s3-buckets-some-useful-tools/), 
que podéis imaginar de qué trata.
		
En resumen, no es que la seguridad por ocultación sea mala seguridad, simplemente no es seguridad: es esperanza y la
esperanza, en seguridad, nunca ha sido una buena estrategia.
