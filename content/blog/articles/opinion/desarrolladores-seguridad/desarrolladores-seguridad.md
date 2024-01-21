+++
title = "¿Por qué los desarrolladores no se preocupan por la seguridad y por qué deberían?"
date = "2019-10-11"
description = "La seguridad informática es uno de los problemas más importantes de cualquier desarrollo pero ¿realmente los desarrolladores se preocupan por ello? ¿deberían hacerlo?"
thumbnail = "/articles/opinion/desarrolladores-seguridad/desarrolladores-seguridad.jpg"
tags = [ "Opinión", "Seguridad" ]
+++

Este fin de semana, raro en mí, no tenía ningún proyecto de programación en mente excepto la documentación
de [BauPlugStudio](http://bauplugstudio.webs-interesantes.es/), es decir, 
era el fin de semana perfecto para procastinar y descansar un poco la mente evitando una tarea desagradable.
	
El caso es que, no sé muy bien cómo, pasé de escuchar la canción [The day of the routers died](https://www.youtube.com/watch?v=M6OJYR_p7aU) a
ver conferencias de [Chema Alonso](https://www.youtube.com/user/Chemai64)
sobre **seguridad informática**.
	
Aparte de comentar cuál ha sido el camino seguido para pasar de un tema a otro, que nos llevaría a escribir otro artículo, 
las conferencias me hicieron reflexionar sobre la seguridad informática y cómo los desarrolladores nos enfrentamos a ella. 
	
En mi caso, es bastante sencillo: olvidemos que existe el problema y confiemos en que el sistema esté suficientemente 
actualizado para evitar las brechas de seguridad, es decir, traslademos el asunto al departamento de sistemas que para eso está.

Sin embargo, que los desarrolladores no tengamos en cuenta los aspectos básicos de seguridad es un asunto muy serio sobre el que deberíamos
examinarnos a conciencia. ¿Por qué el programador no se procupa la seguridad de su sistema y por qué debería
hacerlo en cualquier caso?
	
En mi caso, como todos saben, soy uno de esos desarrolladores eternamente desplazado a un cliente,
es decir, un 'consultor externo' que queda bonito en el curriculum pero al final consiste en 'haz lo que te decimos'. 
En ese entorno en el que la decisión técnica sobre el desarrollo corresponde al cliente, el análisis acaba siendo 
'lo quiero bonito y barato'. La parte de 'bueno' que falta en la frase, cae dentro de una categoría que va 
entre 'si te da tiempo' y 'si no entorpece lo demás' y justamente después de 'quiero el botón con fondo en rojo' 
que suele ser infinitamente más importante.
	
Eso hace que las secciones del desarrollo relacionadas con la infraestructura y la arquitectura sean consideradas una
pérdida completa de tiempo, recursos y dinero. No sólo hablo de seguridad, en este campo se incluyen temas como la gestión de requisitos,
la documentación de cualquier tipo, las pruebas... De hecho, si quieres hacer algo tan sencillo como una gestión de log más te vale
que lo hagas de forma que no entorpezca el desarrollo y a ser posible que nadie se entere que lo estás haciendo.
	
En segundo lugar, entre los desarrolladores hay un profundo desconocimiento de los temas de seguridad. Si preguntas qué significan
términos como inyección de SQL o [MD5](Artículos\Criptografía\Obtener el valor MD5 de una cadena) u 
[oAuth](Artículos\Criptografía\Qué es oAuth), certificados digitales, etc... seguramente obtengas cara de
desconcierto o el consabido 'ese es un problema de sistemas'. Si vas a preguntas más complicadas como la diferencia entre WPA y WPA2,
qué es [IPSec](http://es.wikipedia.org/wiki/IPsec"), 
cuál es la mejor forma de asegurar una VPN o qué es DNS Hijacking te arriesgas además a una mala contestación. 
	
Y me incluyo: tampoco sé exactamente cuál es la diferencia entre WPA y WPA2 pero prometo consultarlo (... otro día).

Porque no lo neguemos, la seguridad informática es un tema arduo, complicado y tremendamente aburrido para el que no tenemos formación
y que requiere una disciplina que muchas veces no seguimos.	Si unimos a esta falta de información y tiempo la despreocupación del cliente 
llegamos a la situación actual que podríamos resumir como: 'sistemas esperando un fallo'.

Los desarrolladores tenemos un problema añadido y es que trabajamos con usuarios que no quieren saber nada de seguridad
y a quienes algo tan sencillo como las contraseñas les resultan molestas. De hecho, he tenido que trabajar con jefes de desarrollo que entre
sus requisitos incluyen que las claves no tengan más de cuatro caracteres, todos numéricos a ser posible.
	
Unas preguntas sencillas para reflexionar: ¿cómo almacenáis las claves de los usuarios en vuestra base de datos? 
¿en texto plano? Si es así no es sólo un fallo de seguridad si no que además 
os estáis saltando a la torera la ley de protección de datos. O por ir un poco más lejos ¿cómo transmitís el usuario 
y la clave desde el cliente al servidor? ¿utilizáis algún método para encriptar esa contraseña o simplemente la enviais 
como un campo más dentro de vuestro formulario?
	
Algo más ¿cómo transmitís la información sensible? Es decir, esos listados inmensos de cuentas bancarias entre el servidor
y el cliente supongo que estarán asegurados con https por lo menos. 
	
**Nota:** Siempre cuento una anécdota que es desgraciadamente cierta, hace unos doce años solicité al departamento de desarrollo de una 
empresa de telefonía (que ya no existe) algunos datos para pruebas de una aplicación que les estábamos programando y me enviaron
'por razones de seguridad': un disco de 3,5 pulgadas con un archivo Excel comprimido en un ZIP con contraseña. Ese archivo
contenía 30.000 clientes reales incluyendo su nombre, DNI, fecha de nacimiento y número de cuenta bancaria. 
Afortunadamente soy honrado, pero el disco pasó por mensajeros, secretarias y administrativos antes de llegar 
a mi mesa. Claro que eran otros tiempos... eso ya no ocurre, para eso tenemos el correo electrónico que, como todos sabemos,
es mucho más seguro.
	
Hablemos de derechos, supongo que conocemos la diferencia entre autentificación y autorización. Una vez autentificado vuestro
usuario ¿cómo sabéis a qué partes de vuestra aplicación accede? ¿habéis tenido en cuenta los aspectos básicos de elevación
de privilegios? ¿consultáis vuestros sistemas de traza para asegurar que tenéis controlados los accesos no permitidos?
	
Y dejemos el desarrollo por un momento: ¿habéis informado a los usuarios que su contraseña es personal e intransferible? 
¿estáis seguros que los ordenadores, móviles y tabletas que utilizan para acceder a los datos desde su casa o la oficina
están convenientemente actualizados? ¿evitáis que accedan desde dispositivos móviles a redes WiFi públicas?

Fijaos que estoy hablando de la parte más básica de cualquier sistema: acceso. No
estamos hablando de errores que un cibercriminal pueda aprovechar si no de todas aquellas cosas que deberían estar en nuestro
manual de usos y costumbres cada vez que arrancamos una aplicación. La pregunta es ¿lo estamos haciendo?
	
No te excuses pensando que estás trabajando en una empresa pequeña y que tus datos no son importantes. Haz un experimento 
sencillo: conecta un ordenador a Internet sin ningún dato ni aplicación excepto un sniffer para 
observar las comunicaciones y espera un rato. ¿Ves la cantidad de tráfico que se genera? ¿no se te ha ocurrido preguntarte quién 
se está comunicando con ese ordenador y para qué? 
	
Ten en cuenta que esas amenazas contra tu seguridad no solo la generan cibercriminales de nivel 10: también son personas 
con acceso a aplicaciones de búsqueda de contraseñas, sniffers, herramientas de denegación de servicio y un largo etcétera 
accesible a todo el que tenga la mínima curiosidad. 
	
Por eso cada día es más importante la seguridad. Cada vez más, vivimos en un entorno donde todo 
está conectado y dónde la información se traduce en dinero y cualquier dato es aprovechable: 
tu número de identificación o DNI, tu fecha de nacimiento, tu número de cuenta bancaria. 
Los hay más o menos peligrosos pero si decimos que son datos protegidos es por algo. 
	
Por resumir, aquello en lo que no pensamos los desarrolladores es lo que utilizan los cibercriminales para entrar en 
nuestros sistemas y obtener información y, como dice Chema Alonso en alguna de sus conferencias, lo peor es que no vamos 
a tener una segunda oportunidad: una vez que un cibercriminal ha entrado en tu sistema olvídate de tus secretos. 
A partir de ese momento es público, no vas a recuperarlo, lo máximo que puedes hacer es pedir disculpas.

Para terminar, repite conmigo: prometo hacer las cosas mejor en el futuro, prometo hablar con mis jefes para que se incluya la seguridad
como requisito indispensable, prometo estudiar las bases de seguridad informática, prometo
leer un par de libros sobre hacking para aprender cuáles son las técnicas a las que debo enfrentarme, prometo pelearme ligeramente con
los algoritmos de criptografía, prometo revisar mi código, prometo dejar poca superficie de ataque en mis desarrollos, prometo consultar
si hay copias de seguridad de mis bases de datos, prometo solicitar que se instale una DMZ en el sistema, prometo reducir los privilegios
de ejecución de mis aplicaciones al mínimo... bueno, si tengo tiempo.