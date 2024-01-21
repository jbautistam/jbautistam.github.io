+++
title = "Mis hallazgos, involuntarios, con Shodan"
date = "2019-10-11"
description = "Lo que puedes llegar a encontrarte cuando navegas por Shodan"
thumbnail = "/articles/seguridad/hallazgos-shodan/hallazgos-shodan.jpg"
tags = [ "Seguridad" ]
+++

Una de las herramientas que me encuentro de vez en cuando escuchando conferencias de seguridad
es [Shodan](https://www.shodan.io/).
	
Para quien no lo conozca, Shodan es un buscador de dispositivos en Internet. Por comparación es el
Google de los dispositivos aunque no indexa páginas HTML si no máquinas y puertos.

Es curioso porque nos permite, por ejemplo, buscar los servidores que hay por el mundo con **Sql Server** 
instalado, los que tienen abierto el puerto 21 para comunicación por FTP, las WebCam... vamos, lo que
se os ocurra.
	
El otro día, a los dos minutos de navegación casual (juro que no lo estaba buscando), me encontré con un servidor 
curioso (por supuesto las direcciones están cortadas):

![Servidor](/blog/articles/seguridad/hallazgos-shodan/servidor.jpg "Servidor")
	
Lo que me sorprendió fue el puerto 9000 o mejor dicho, lo que encontré en la definición de ese puerto:

![Resultado del puerto 9000](/blog/articles/seguridad/hallazgos-shodan/puerto-9000.jpg "Resultado del puerto 9000")
	
Así, a simple vista, captó mi atención la línea **Octoprint**. Para quien no lo sepa, 
[OctoPrint](http://octoprint.org/) es un interface Web de código abierto para impresoras 3D:
	
![Octoprint](/blog/articles/seguridad/hallazgos-shodan/octopring.jpg "Octoprint")	
	
La curiosidad, como se suele decir, mató al gato y a mí me llevó a pulsar sobre el enlace de **Shodan**, sin mala intención,
el pensamiento era *no serán capaces...*. Pues sí. Lo eran:

![Consola de la impresora](/blog/articles/seguridad/hallazgos-shodan/impresora.jpg "Consola de la impresora")

Para el que se pregunte qué está viendo, lo que tenemos es la página de control de una impresora 3D de la República Checa.
Vemos el estado de la impresora, su temperatura, la terminal o los últimos documentos enviados a la impresora y su estado de impresión.
	
De hecho, podemos descargarnos los documentos que se han impreso ¿no os lo creéis? 

![Documento de la impresora](/blog/articles/seguridad/hallazgos-shodan/documento-impresion.jpg "Documento de la impresora")
	
¿A qué viene todo esto? Llevamos algún tiempo hablando de la mala seguridad del IoT y de lo poco que le interesa y sabe el usuario 
sobre seguridad. Lo que tenemos es una empresa que ha conectado una impresora 3D a la red y le ha dado acceso a todo el mundo.
Cualquiera puede ver lo que imprime y posiblemente con un poco de maña también podrá utilizar esa impresora para lo que quiera.
	
Por supuesto, me puse en contacto con la empresa y en mi mal inglés y con la inestimable ayuda del traductor de Google, 
les expliqué que tenían un problema. 
	
No me han respondido. Puede que haya pasado a su bandeja de spam, que me consideren un ciberdelincuente y se estén planteando una denuncia o
que simplemente hayan solucionado el problema y hayan decidido no dar las gracias. En cualquier caso, creo que es lo que hay que hacer.
	
Y esto nos lleva a una cita mítica en esto de la seguridad:

> No es necesario que te preocupes por tu seguridad. Alguien lo hará.

En este caso he sido yo pero alguien debería poner un aviso a los usuarios en los dispositivos diciendo que si los conectan a la red
se exponen a este tipo de cosas. De hecho, me uno a la petición de 
[Troy Hunt](https://www.troyhunt.com/what-would-it-look-like-if-we-put-warnings-on-iot-devices-like-we-do-cigarette-packets/)
para poner anuncios en los dispositivos del mismo modo que los ponemos en las cajetillas de cigarrillos aunque espero
que no con el mismo resultado.

![avisos-seguridad.jpg](/blog/articles/seguridad/hallazgos-shodan/avisos-seguridad.jpg "avisos-seguridad.jpg")