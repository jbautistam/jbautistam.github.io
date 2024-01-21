+++
title = "¿Qué es oAuth?"
date = "2019-10-11"
description = "¿Qué es el protocolo oAuth? En este artículo te ofrecemos una introducción a este protocolo de autentificación"
thumbnail = "/articles/seguridad/que-es-oauth/que-es-oauth.jpg"
tags = [ "Programación", "Seguridad" ]
+++

A poco que hayáis intentado programar librerías que utilicen las APIs de Twitter, Facebook o Google posiblemente 
os hayáis tropezado con el término OAuth.

Pero, ¿qué es oAuth?

En pocas palabras, oAuth es un protocolo de autentificación y autorización centralizado y anónimo, es decir, un 
protocolo que nos permite identificar un usuario y autorizar el acceso a cierta aplicación sin almacenar realmente 
los datos del usuario.

Dicho así, puede parecer un galimatías, es decir, ¿cómo podemos autorizar un usuario sin saber quién es realmente y sin conocer sus datos?

Para explicarlo bien, vamos a hacer un poco de historia. Hasta ahora, si deseábamos autentificar un usuario necesitábamos un código de 
inicio de sesión o login y una contraseña. Nuestra aplicación era la encargada de almacenar estas claves y mantenerlas en un lugar 
seguro. Con la llegada de Internet comenzaron a surgir una gran cantidad de sitios y aplicaciones que solicitaban un registro 
para utilizarlas. Esto obliga a nuestros usuarios a recordar cada una de las contraseñas de cada sitio.

El gran problema es que con tal cantidad de aplicaciones los usuarios comienzan a utilizar la misma contraseña para todo con los 
riesgos de seguridad que provoca algo así. Si para evitar esto habilitamos algún sistema de caducidad de contraseñas, en lugar 
de proporcionar una solución conseguimos agravar el problema y obligamos al usuario a recordar una clave más o a utilizar el 
botón de "He olvidado mi contraseña".

La mejor solución es intentar crear un sistema centralizado que nos permita acceder a todos nuestros sitios desde un único lugar. 
Para ello en los últimos años han surgido varios protocolos con mayor o menor suerte hasta que apareció oAuth 
que parece que ha venido para quedarse por el apoyo que está recibiendo de los grandes dominadores de la Web como Facebook, 
Twitter o Google.

Pero, ¿cómo funciona este sistema?

En primer lugar, como decíamos al principio, oAuth es un sistema centralizado, es decir, existe un servidor encargado de 
almacenar las cuentas de usuario y permitirles el acceso. En este caso, el protocolo permite que cualquier sitio se pueda 
convertir en un proveedor de acceso pero siempre podemos utilizar uno de los más conocidos, como por ejemplo Twitter.

Una vez sabemos quién se encarga de mantener las cuentas, aparece un segundo concepto que es el de aplicación, aunque con su 
sentido más amplio, es decir, no sólo con su significado de programa de escritorio o móvil si no también englobando los sitios 
Web. Es decir, si deseamos acceder a un servidor de oAuth, debemos crearnos una cuenta sobre el proveedor (digamos Twitter) y 
solicitar una clave de aplicación.

Al crear una aplicación en nuestra cuenta, el proveedor nos proporcionará dos códigos, ConsumerKey y ConsumerKeySecret. Estos dos 
códigos se deben mantener en sitio seguro pues son los que permitirán el acceso posteriormente. Podemos pensar que son similares
a la clave pública y privada en la [criptografía de clave pública](/blog/articles/seguridad/clave-publica/clave-publica). 

Una vez tenemos nuestros códigos de aplicación, cada vez que deseemos autentificar a un usuario, lo que haremos será delegar 
esta autentilficación al proveedor (siguiendo con el ejemplo anterior a Twitter). Para ello, lo normal es hacer una llamada 
a una página especial del proveedor indicando las claves de nuestra aplicación. El usuario puede en este momento introducir 
su usuario y contraseña autorizando a nuestra aplicación o bien denegar el acceso. En caso que nos autorice, el proveedor nos 
devuelve un token de acceso temporal o permanente que podemos almacenar y que a partir de ese momento identifica unívocamente 
la aplicación y el usuario. Es decir, cada vez que deseemos acceder al proveedor podemos utilizar nuestra clave de aplicación 
y el token de acceso devuelto para autentificar el usuario sin necesidad de solicitar de nuevo su usuario y contraseña. Por 
supuesto, todo este protocolo de autentificación utiliza un sistema de seguridad basado en sistemas criptográficos que 
evitan que las claves privadas se transmitan en texto plano entre el cliente y el servidor.

Hay que recordar que el usuario y la contraseña en realidad se solicitan sobre la página del proveedor de acceso, Twitter 
para entendernos, es decir, nuestra aplicación nunca conoce la clave del usuario, ni siquiera su código de inicio de sesión. 
Así conseguimos el acceso anónimo que pretendíamos en un principio.

Al mismo tiempo, al utilizar un servidor de acceso, volvamos a decir Twitter, los usuarios tienen siempre el control de su cuenta 
y las aplicaciones que tienen acceso a ellas. Esto proporciona a los usuarios finales ciertas ventajas:

1. Sólo tienen que recordar una contraseña, por difícil que sea, para acceder a todas sus aplicaciones o sitios.
2. Si su contraseña caduca o desean cambiarla pueden hacerlo sin problemas dado que las aplicaciones no utilizan éstas si no 
que guardan los códigos o tokens de acceso que permanecen inmutables.
3. El proveedor de acceso mantiene una lista de aplicaciones autorizadas. El usuario en cualquier momento puede denegar el 
acceso a cualquier aplicación.
4. El usuario tiene la seguridad de no tener su contraseña dispersa en varias aplicaciones y cuentas.

Como programadores, este protocolo también nos proporciona bastantes ventajas como las posibilidades de acceso a las APIs de 
un proveedor o la seguridad que proporcionamos al usuario sin la necesidad de almacenar sus credenciales.

Espero que esta pequeña introducción os haya servido para perder un poco el miedo a oAuth, os recomiendo el 
[sitio oficial de oAuth](http://oauth.net) donde podréis encontrar más información sobre el 
tema. En breve prometo un segundo artículo en el que explicaré cómo programar una librería de acceso a oAuth utilizando C#.