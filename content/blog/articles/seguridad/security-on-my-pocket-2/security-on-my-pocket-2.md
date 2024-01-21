+++
title = "Security on my Pocket 2"
date = "2019-10-11"
description = "Relación de temas de seguridad encontrados los últimos dos meses"
thumbnail = "/images/noimage.jpg"
tags = [ "Seguridad" ]
+++

 Continúo con mi afán de liberar mi lista de enlaces de Pocket sobre seguridad.
 
Como siempre, la falta de tiempo para clasificar y documentar los enlaces me ha dejado un saco bastante lleno de novedades
y catástrofes. Vamos a ver si lo ponemos un poco en orden.
 	
2018 empezó fuerte: Meltdown y Spectre nos atacaron por sorpresa a principios de año. El problema esta vez afectaba directamente
a los procesadores y al sistema de acceso a memoria compartida primero de Intel y después de AMD y obligaba a parchear todos 
los sistemas operativos.
 	
Aparte de las declaraciones de [Linus Torvalds](https://techcrunch.com/2018/01/22/linus-torvalds-declares-intel-fix-for-meltdown-spectre-complete-and-utter-garbage/)
sobre los arquitectos de Intel:
 	
> Has anybody talked to them and told them they are f*cking insane?
 
que supongo que a nadie sorprendieron, una de las mejores explicaciones que encontré sobre la vulnerabilidad fue ésta
[Meltdown and Spectre](https://meltdownattack.com/).

Quizá no tenga mucho que ver, pero en noviembre del año pasado Intel ya tuvo sus primeros problemas:
[Fallos en el firmware de Intel dejan vulnerables millones de equipos sin importar qué sistema operativo usen](https://www.genbeta.com/seguridad/fallos-en-el-firmware-de-intel-dejan-vulnerables-millones-de-equipos-sin-importar-que-sistema-operativo-usen).

## Navegadores, botnets, criptomonedas y similares

No sé si es que estos meses han sido especialmente malos para los navegadores o si comenzamos a ser mucho más críticos con este
tipo de sistemas pero desde luego no nos han faltado noticias y nuevos ataques.
	
La estrella está siendo el minado de criptomonedas. Tras las noticias de la aparición de scripts maliciosos en algunos sitios
importantes (entre ellos el de Telefónica), saltó la noticia de la aparición de este malware en la propia
Web de [YouTube](https://arstechnica.com/information-technology/2018/01/now-even-youtube-serves-ads-with-cpu-draining-cryptocurrency-miners/).
	
Lo bueno es que basta con cerrar la pestaña del navegador para que deje de ejecutarse el script de minado. Bueno, últimamente
esto tampoco sirve: [Cryptocurrency Mining Scripts Now Run Even After You Close Your Browser](https://thehackernews.com/2017/11/cryptocurrency-mining-javascript.html).
Ya tenemos scripts que siguen ejecutando incluso cuando cerramos las ventanas.
	
Por supuesto, la privacidad continua siendo uno de los grandes defectos de los navegadores. En estos días que tus datos
personales son tan importantes, surgen sistemas para
[identificar a los usuarios](http://www.microsiervos.com/archivo/seguridad/sitios-web-identificar-usuarios.html) incluso aunque utilicen diferentes navegadores.

Continuando con los peligros de las extensiones del navegador, comienza la lucha por evitar su detección. En concreto, algunas
extensiones maliciosas para [Chrome y Firefox](https://www.neowin.net/news/rogue-extensions-that-hijack-chrome--firefox-are-near-impossible-to-remove) 
toman el control del sistema de eliminación de extensiones de forma que sean prácticamente imposibles de eliminar para un usuario sin conocimientos avanzados.
	
No sé si esto se convertirá en tendencia, pero comienzan a ser una norma los artículos sobre los peligros de las librerías
descontroladas de JavaScript de las que hemos hablado en algunas ocasiones. Por ejemplo:
	
* [I’m harvesting credit card numbers and passwords from your site. Here’s how](https://hackernoon.com/im-harvesting-credit-card-numbers-and-passwords-from-your-site-here-s-how-9a8cb347c5b5) 
describe cómo se podría llegar a robar información de sitios Web con inyección de código en librerías 'influyentes'.

* [Browser as Botnet, or the Coming War on Your Web Browser](https://medium.com/@brannondorsey/browser-as-botnet-or-the-coming-war-on-your-web-browser-be920c4f718) 
describe cómo transformar los navegadores en las nuevas botnet utilizando los sistemas de publicidad. Muy interesante. Me extraña que no se nos haya ocurrido antes.
	
* [CoffeeMiner: ataque MitM para minar criptomonedas](https://blog.segu-info.com.ar/2018/01/coffeeminer-ataque-mitm-para-minar.html) 
de lo mejorcito que he leído últimamente: mezclar ataques Rogue AP con inyección de script para minar criptomonedas. Y con código fuente en Github para los
más atrevidos.
	
* [77% of 433,000 Sites Use Vulnerable JavaScript Libraries](https://snyk.io/blog/77-percent-of-sites-still-vulnerable/) 
que nos podemos imaginar de qué trata.
		
Y al hilo de las botnets y las criptomonedas, nos encontramos esto:
[New botnet infects cryptocurrency mining computers, replaces wallet address](https://arstechnica.com/information-technology/2018/01/in-the-wild-malware-preys-on-computers-dedicated-to-mining-cryptocurrency/), 
que avisa de la aparición de nuevos botnets para minar este tipo de monedas, algo que por cierto ha hecho que caiga el valor de alguna de ellas
como Ethereum.
 	
## Dispositivos, redes e IoT

El día de Reyes me desperté y entre mis regalos tenía una maravillosa unidad de backup de Western Digital. Desgraciadamente,
también recibí por Twitter un enlace a este artículo: 
[Critical Unpatched Flaws Disclosed In Western Digital 'My Cloud' Storage Devices](https://thehackernews.com/2018/01/western-digital-mycloud.html) 
en el que desglosaban las vulnerabilidades de estos dispositivos incluyendo una de esas puertas traseras imposibles de parchear.
	
Lo de las puertas traseras quizá deberíamos hacérnoslo mirar, por ejemplo Lenovo incluía una para su 
[Lector de huellas digitales](http://www.theregister.co.uk/2018/01/26/lenovo_thinkpad_fingerprint_manager_vulnerability/) 
que deja expuestos todos los datos almacenados del sistema.
		
Y siguiendo con el tema, hay quien encuentra sistemas de minería de criptomonedas con la clave por defecto root / root:

![Bitcoin miners](/blog/articles/seguridad/security-on-my-pocket-2/bitcoin-miners.jpg "Bitcoin miners")

por si acaso no nos habíamos dado cuenta que cripto y seguridad no es lo mismo.

No pensemos que todo este tipo de cosas se solucionan con una buena aplicación de mantenimiento de contraseñas. Si caemos
en la tentación podemos leer este artículo:
[Windows 10 included password manager with huge security hole](https://www.engadget.com/2017/12/16/windows-10-bundled-password-manager-had-security-flaw/) 
que quizá nos pasó algo desapercibido por los problemas de Apple con los usuarios root:
	
![Entrar con el usuario root en Apple sin contraseña](/blog/articles/seguridad/security-on-my-pocket-2/root-apple.jpg "Entrar con el usuario root en Apple sin contraseña")
	
Otras cosas: al parecer, después de las vulnerabilidades descubiertas en WEP, WPA y WPA2, este año se lanzará
[WPA3](https://www.engadget.com/2018/01/08/wifi-alliance-wpa3-standard/), el nuevo estándar
para los sistemas WiFi que promete ser más seguro. No sé si esto ya lo habíais escuchado antes.
	
Por cerrar este tema, por casualidad llegué a esta Web: [Router Security]({https://routersecurity.org/bugs.php)
que lista los errores de seguridad conocidos de muchos routers del mercado. Conviene echarle un vistazo aunque como supondréis, es kilométrica.
	
En cuanto a Android, he recopilado estas noticias:

* [New Android vulnerability allows attackers to modify apps without affecting their signatures](https://www.guardsquare.com/en/blog/new-android-vulnerability-allows-attackers-modify-apps-without-affecting-their-signatures)
* [Severe flaws in most popular programming languages could expose to hack any secure application built on top of them](http://securityaffairs.co/wordpress/66556/hacking/flaws-programming-languages-fuzzing.html)
* [Massive Breach Exposes Keyboard App that Collects Personal Data On Its 31 Million Users](https://thehackernews.com/2017/12/keyboard-data-breach.html)
* [Security Flaw Left Major Banking Apps Vulnerable to MiTM Attacks Over SSL](https://thehackernews.com/2017/12/mitm-ssl-pinning-hostname.html)
* [ParseDroid vulnerabilities threatened most Android development tools](https://www.developer-tech.com/news/2017/dec/07/parsedroid-vulnerabilities-threatened-most-android-development-tools/)
* [Critical Flaw in Major Android Tools Targets Developers and Reverse Engineers](https://thehackernews.com/2017/12/android-development-tools.html)
* [Dozens of companies are tracking you via your Android phone](https://medium.com/@dallincrump/dozens-of-companies-youve-never-heard-of-are-tracking-you-via-your-android-phone-d178c34ebc32)

## El mundo físico

Hace algún tiempo leí un PoC que explicaba cómo entrar en los sistemas de navegación de un avión a partir de la red WiFi de los pasajeros.
Se llegó a demostrar en simuladores pero no en aviones reales. En noviembre se reprodujo la prueba con un avión real en tierra,
concretamente un Boeing 757. Podéis leer el artículo completo en 
[Homeland Security team remotely hacked a Boeing 757](https://www.csoonline.com/article/3236721/security/homeland-security-team-remotely-hacked-a-boeing-757.amp.html).
		
Una de espionaje: [Is Your DJI Drone a Chinese Spy? Leaked DHS Memo Suggests](https://thehackernews.com/2017/12/dji-drone-china-spying.html). 
El primer dron que conozco que envía sus datos a China, algo que también parece que ocurre con los teléfonos de
[OnePlus](http://bgr.com/2018/01/26/oneplus-data-collection-clipboard-app/) aunque la empresa ha negado las acusaciones estos últimos días.	

Por supuesto, los cajeros automáticos (ATM) siguen siendo un objetivo para los ciberdelincuentes y parece que
comienzan a llegar a [Estados Unidos](https://krebsonsecurity.com/2018/01/first-jackpotting-attacks-hit-u-s-atms/)
tras su gira por Europa y Asia.	

## Para desarrolladores
 
En cuanto a todas esas cosas que deberíamos tener en cuenta como desarrolladores o administradores, 
Miguel Angel Arroyo publicaba un estudio sobre cómo localizar servidores MongoDb que hubiesen sido víctimas de 
[ransomware](https://hacking-etico.com/2017/03/01/ransomware-en-mongodb/) utilizando Shodan.
 	
En esta misma Web, mantienen un artículo sobre las [principales vulnerabilidades web](https://hacking-etico.com/2017/04/04/las-principales-vulnerabilidades-web). 
Posiblemente nada que no supiéramos ya, pero siempre viene bien una lista de este tipo para refrescar la memoria.

Y este quizá sea incluso más antiguo pero me sorprendió ver que la web de Martin Fowler también habla de las bases de 
[seguridad Web](https://martinfowler.com/articles/web-security-basics.html). No 
sé porqué si le hacemos caso en cosas como el código limpio, no comenzamos a tenerle en cuenta para aumentar
nuestros conocimientos sobre seguridad.
	
Aunque ya han pasado tres o cuatro meses desde el Codemotion hasta ahora no había visto completa la charla de
[Luis Ruiz Pavón](https://twitter.com/luisruizpavon) sobre seguridad 
[Los ataques web más comúnes en acción](https://www.youtube.com/watch?v=VsDyHANVmcQ&feature=youtu.be). 
Una pena que los últimos minutos se pierda el audio.

Más cosas: releyendo a Troy Hunt, me he encontrado con un artículo interesantísimo con una lista de pasos para configurar 
correctamente HTTPs en nuestros sitios: [The 6-Step "Happy Path" to HTTPS](https://www.troyhunt.com/the-6-step-happy-path-to-https/).
	
Por cierto, como suele pasar a este artículo llegué tras leer este otro:
[I am Sorry You Feel This Way NatWest, but HTTPS on Your Landing Page Is Important](https://www.troyhunt.com/im-sorry-you-feel-this-way-natwest-but-https-on-your-landing-page-is-important/) 
que creo que todos deberíamos leer.
 	
## Y para terminar

Una lista rápida de cosas que se quedaron en el tintero:


* [Wargames](http://overthewire.org/wargames/): si os interesan los conceptos de seguridad, esta Web plantea una serie de ejercicios 
y juegos que nos ayudarán en el aprendizaje.

* [¡Lleno, por favor! Gasolineras](https://hacking-etico.com/2017/01/27/lleno-favor-gasolineras):
ya hemos hablado de [Shodan](/blog/articles/seguridad/hallazgos-shodan) 
alguna vez en esta Web, pero este artículo es incluso más demoledor ¿crees que podrías tomar el control de los tanques de una gasolinera?		

* [Investigando los discos duros de Bin Laden: malware, contraseñas, warez y metadatos](http://blog.elevenpaths.com/2017/11/investigando-los-discos-duros-de-bin.html): 
un gran artículo sobre informática forense, algo de lo que desgraciadamente hablamos poco en esta web pero que la gente de Eleven Paths
explica al detalle.

* [The lax computer security of British MPs - as detailed in their own tweets](https://www.grahamcluley.com/lax-computer-security-british-mps-detailed-tweets/): 
me gusta pensar que si educamos a los usuarios evitaremos muchos problemas de seguridad. Mal vamos cuando ni siquiera a nuestros políticos les interesa la seguridad.

* [DoNotReply](http://www.microsiervos.com/archivo/seguridad/historia-donotreply-com.html):
¿te has preguntado alguna vez a quién le llegan los correos de respuesta a las direcciones '@donotreply.com'? Pues aquí te

lo explican. Cierto, no tiene mucho que ver con la seguridad pero me hizo gracia.
	
Pues ya está. Mi lista de Pocket es un poco más pequeña aunque se han quedado algunas cosas en el tintero. Quién sabe, quizá el próximo mes.