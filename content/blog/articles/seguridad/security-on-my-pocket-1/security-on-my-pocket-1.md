+++
title = "Security on my pocket  - 1"
date = "2019-10-11"
description = "Artículos interesantes sobre in/seguridad "
thumbnail = "/articles/seguridad/security-on-my-pocket-1/security-on-my-pocket-1.jpg"
tags = [ "Seguridad" ]
+++

Hace unos meses, después de escribir el artículo
[¿por qué a los desarrolladores no les preocupa la seguridad?](/blog/articles/opinion/desarrolladores-seguridad/desarrolladores-seguridad)
comencé a preparar una charla sobre el tema y me acostumbré a almacenar artículos de seguridad en mi Pocket.
	
Sigo manteniendo esa costumbre y como resultado, mi lista de marcadores crece y crece haciéndose
algo inmanejable. Se me ha ocurrido compartir esa lista sobre vulnerabilidades en un artículo que posiblemente
se transforme en una serie a lo largo del tiempo al mismo tiempo que aprovecho para hacer limpieza.
	
Estos artículos son los que he encontrado estas últimas semanas, por supuesto no están todos los que son, simplemente
los que más me han llamado la atención.	

## Ataques a empresas

Una de las últimas empresas que ha saltado a las noticias por perder datos de sus clientes ha sido
Equifax. Y siguen surgiendo noticias sobre el tema

* [Equifax Was Warned](https://motherboard.vice.com/en_us/article/ne3bv7/equifax-breach-social-security-numbers-researcher-warning): 
lo habitual: avisas a una empresa de un fallo en el sistema y nadie te hace caso.

* [Equifax reportedly used 'admin' as password in Argentina](https://www.cnet.com/news/equifax-argentina-vulnerability-admin/): 
que como digo siempre, al menos no era '1234'.

## Botnets, bots y similares

Desde que apareció Mirai y se liberó su código fuente en Internet, las botnets sobre dispositivos IoT se han
convertido en un problema preocupante. 
	
Ahora tenemos aquí **IoT reaper** que no sólo busca dispositivos
con claves por defecto como Mirai si no que además busca vulnerabilidades conocidas en los dispositivos
logrando una red de botnet incluso más extensa:
	
* [New Mirai-Linked IoT Botnet Emerges](http://www.securityweek.com/new-mirai-linked-iot-botnet-emerges)
* [Researchers warn of new botnet that could take down the internet](https://www.techspot.com/news/71514-researchers-warn-new-botnet-could-take-down-internet.html)
* [Botnet IoT crece en las sombras durante el mes de septiembre](http://unaaldia.hispasec.com/2017/10/botnet-iot-crece-en-las-sombras-durante.html)
* [Attackers Use DDoS Pulses to Pin Down Multiple Targets](https://www.incapsula.com/blog/pulse-wave-ddos-pins-down-multiple-targets.html)

Y hablando de bots y siguiendo con el temita de los rusos influyendo en las votaciones apareció esta otra noticia: 

[Descubren una red de bots en Twitter que podría haber influido en la votación del Brexit](https://www.genbeta.com/redes-sociales-y-comunidades/descubren-una-red-de-bots-en-twitter-que-podria-haber-influido-en-la-votacion-del-brexit)

## Móviles, redes y ataques a usuarios

Las redes móviles no dejan de darnos disgustos, aparte de KRACK para WPA2, aparece DUHK, un sistema para
atacar routers a partir de las contraseñas almacenadas en el código.

* [WPA2: Broken with KRACK. What now?](https://www.alexhudson.com/2017/10/15/wpa2-broken-krack-now/)
* [Attack of the week: DUHK](https://blog.cryptographyengineering.com/2017/10/23/attack-of-the-week-duhk/)

Y por supuesto no podían faltar los clásicos enlaces de aplicaciones vulnerables:
 
* [Tinder, Ok Cupid, Badoo y otras apps para ligar podrían exponer mensajes y ubicaciones](https://www.genbeta.com/seguridad/tinder-ok-cupid-badoo-y-otras-apps-para-ligar-podrian-exponer-mensajes-y-ubicaciones)

* [XNSPY: Un Spyware para iOS nuevo en la ciudad](http://www.seguridadapple.com/2017/09/xnspy-un-spyware-para-ios-nuevo-en-la.html)

* [Lo que faltaba, apps en Android y botnets que nos quieren usar para minar criptomonedas](https://www.genbeta.com/seguridad/lo-que-faltaba-apps-en-android-y-botnets-que-nos-quieren-usar-para-minar-criptomonedas)

* [Hackers Can Steal Windows Login Credentials Without User Interaction](https://www.bleepingcomputer.com/news/security/hackers-can-steal-windows-login-credentials-without-user-interaction/): 
los archivos SCF de nuevo.

* ['Combosquatting' attack hides in plain sight to trick computer users](https://phys.org/news/2017-10-combosquatting-plain-sight-users.html)

* [A surge of sites and apps are exhausting your CPU to mine cryptocurrency](https://arstechnica.com/information-technology/2017/10/a-surge-of-sites-and-apps-are-exhausting-your-cpu-to-mine-cryptocurrency)
	
Una nota sobre la charla de seguridad: en ella surgió el tema de los ataques a dispositivos móviles suplantando
una WiFi abierta. Surgieron dudas sobre si esto era posible o no y si el móvil almacenaba además del
SSID la mac address del router. Bueno, pues desgraciadamente y si no me equivoco, sí se puede:

* [Rogue AP en Windows con Intel My Wifi](http://www.elladodelmal.com/2009/06/rogue-ap-en-windows-con-intel-my-wifi.html)
* [Montando un Rogue AP con Kali](http://www.securitybydefault.com/2013/11/montando-un-rogue-ap-con-kali.html)
 
## El mundo físico

La separación entre mundo virtual / físico cada vez es más fina y desgraciadamente me voy acostumbrando a este tipo de noticias:
	
* [Security flaw could have let hackers turn on smart ovens](https://phys.org/news/2017-10-flaw-hackers-smart-ovens.html): 
y no sólo eso, si revisamos las noticias veremos neveras, televisores, coches e incluso aviones.

* [The U.S Power Grid Isn’t Prepared for Cyberattacks](https://futurism.com/72874-2/)
* [Sistemas de comunicación marítima con vulnerabilidades típicas de hace una década](http://unaaldia.hispasec.com/2017/10/sistemas-de-comunicacion-maritima-con.html): 
¿alguien adivina? Inyección SQL y ejecución de código con privilegios de administración con una cuenta 'oculta' de administrador. 

## Para desarrolladores

Esta semana me he encontrado con una Web muy interesante que lista los errores de software más comunes
que pueden dejar abiertas brechas de seguridad, por ejemplo podemos encontrar el ranking del 2011
[2011 CWE/SANS Top 25 Most Dangerous Software Errors](http://cwe.mitre.org/top25/).
	
Desgraciadamente parece que se quedaron sin ganas de continuar editando el ranking en el 2013 o bien soy incapaz
de encontrarlo. Si alguien tiene más información sobre el tema se lo agradecería.
	
Y siguiendo con el tema de 'las librerías no actualizadas son un problema de seguridad en el desarrollo' me
he encontrado con esta noticia sobre JavaScript 
[Security vulnerabilities in JavaScript libraries are hard to avoid](https://sdtimes.com/security-vulnerabilities-javascript-libraries-hard-avoid/), 
que por cierto no me sorprende.

## Los enlaces tontos de la semana

Por mucho que hablemos de seguridad, también hay ciertas noticias que nos hacen sonreir de cuando en cuando:

* [Trabajador de la NSA filtra software espía a los rusos por estar pirateando Office](https://www.genbeta.com/actualidad/trabajador-de-la-nsa-filtra-software-espia-a-los-rusos-por-estar-pirateando-office):
para que nos demos cuenta que en todas partes cuecen habas. Al parecer alguien buscaba un keygen para Office
y acabó entregando su sistema a los rusos. Cosas que pasan.

* [What Would It Look Like If We Put Warnings on IoT Devices Like We Do Cigarette Packets?](https://www.troyhunt.com/what-would-it-look-like-if-we-put-warnings-on-iot-devices-like-we-do-cigarette-packets/):
la web de **Troy Hunt** siempre es una mina para encontrar artículos interesantes sobre seguridad. Esta vez se pregunta
que pásaría si en los dispositivos IoT pusiéramos fotos como en los paquetes de cigarrillos.

Y eso es todo por ahora, prometo seguir limpiando mi Pocket de vez en cuando, aún queda mucho en el fondo de la bolsa.