+++
title = "BauMessenger: implementación del protocolo XMPP"
date = "2019-10-11"
description = "Aplicación de prueba para mensajería utilizando XMPP / Jabber"
thumbnail = "/articles/source-code/baumessenger/baumessenger.jpg"
tags = [ "Comunicaciones", "Programación" ]
+++

Los más veteranos del lugar quizá recuerden que sobre el año 2.000 surgieron una nueva línea de aplicaciones
conocidas con el nombre genérico de 'aplicaciones de mensajería instantánea'. Entre estas aplicaciones estaban 
**Icq** (de Mirabilis), **Yahoo! Messenger** (de Yahoo, obviamente) o **MSN Messenger** (de Microsoft) y 
plantearon una nueva guerra por captar usuarios similar a la que se había vivido entre los navegadores.

La función de estas aplicaciones eran permitir enviar información de presencia (el usuario está conectado y libre para conversar o no) y 
mensajes y archivos instantáneamente entre usuarios. Algunos de ellos incluso permitían conversaciones en grupo o chat. 
Todas ellas utilizaban software de comunicación propietario, es decir, no existía un estándar y era imposible contactar 
con un usuario a menos que se utilizase la misma aplicación. ¿Os suena de algo? ¿Alguien ha pensado en WhatsApp / Telegram / Line...?
	
Para evitar estos inconvenientes surgieron aplicaciones como Trillian que permitían comunicar usuarios de diferentes redes aunque
se perdían ciertas funcionalidades en algunas de ellas. Al mismo tiempo, surgió un protocolo que pretendía ser un nuevo
estándar de comunicaciones llamado [Jabber](http://www.jabber.org/) que posteriormente
se convertiría en estándar de IETF bajo el nombre [XMPP](http://xmpp.org/) 
(Extensible Messaging and Presence Protocol - Protocolo extensible de mensajería y presencia).
	
Aunque los programas de mensajería poco a poco fueron cayendo en el olvido relegados por las redes sociales, **XMPP** sigue
funcionando y es la base de aplicaciones conocidas como **Google Talk** o el propio 
[WhatsApp](https://es.wikipedia.org/wiki/WhatsApp).
	
**XMPP** es un prococolo de comunicación abierto (cualquiera puede utilizarlo para sus aplicaciones de mensajería) y extensible:
se puede ampliar para incluir funcionalidades propias. De hecho, si miráis el protocolo veréis extensiones para
prácticamente todo: grupos, envío de archivos, avatares, formularios, suscripción de contenido e incluso videoconferencia.
	
Al ser un protocolo abierto no sólo existen varios clientes si no diferentes servidores la mayor parte de ellos de código libre.
Estos clientes y servidores se comunican entre sí sin ningún tipo de problema: no me hace falta tener una cuenta en el servidor
**jabber.org** para comunicarme con un usuario del servidor **otroservidor.org**. 
	
Además, al ser un protocolo extensible basado en XML, se pueden crear mensajes adicionales (los conocidos como mensajes **x**) 
que implementen nuevas funcionalidades. La mayor parte de las extensiones de las que hablábamos anterioremente se crean a 
partir de este tipo de mensajes.
	
Todas estas razones convierten a **XMPP** en un protocolo que me lleva atrayendo ya algún tiempo y que he utilizado en un par
de proyectos. Siempre me ha sorprendido que no se utilice más a menudo.
	
Lo malo es que el protocolo evoluciona y mi librería base comienza a ser, por decirlo suavemente, un batiburrillo de ideas que 
dificulta enormemente el mantenimiento, por eso decidí buscar código algo más 'estándar' que me sirviera como base para futuras 
aplicaciones.
	
Existen varias librerías de código abierto para **XMPP** en C# en **GitHub** y otros servicios. Por mi parte escogí 
[Sharp.Xmpp](https://github.com/pgstath/Sharp.Xmpp) simplemente porque me pareció que
estaba más actualizada que el resto y se mantenía con cierta regularidad. 
	
Lo malo es que la documentación y los ejemplos de la librería eran por así decirlo, exíguos. Un par de ejemplos de conexión y
carga del roster o lista de contactos pero nada sobre registro de nuevos usuarios. Así que poco a poco me fui haciendo mi 
propia aplicación de mensajería (BauMessenger) y aquí estamos.
	
Por si os interesa y no queréis leer el resto del artículo dedicado a explicar su funcionamiento, el código fuente de la 
aplicación lo podéis encontrar en [GitHub](https://github.com/jbautistam/BauXmppMessenger).

## Librería de comunicaciones

Para trabajar con la librería **Sharp.Xmpp** pero sin 'atarme' a su implementación, decidí desarrollar una librería intermedia
(`LibXmppClient`) que oculta las funcionalidades de Xmpp tras una pequeña capa de dominio.
	
**Nota:** Me he encontrado con algunos problemas al utilizar la versión de **Nuget** de Sharp.Xmpp, por
eso en la aplicación he enlazado directamente la versión del código fuente que estaba utilizando.	
	
La clase que vamos a utilizar de esta librería para mantener las comunicaciones es `JabberManager` (lo siento, siempre
he preferido el término Jabber al de Xmpp, llámenme nostálgico). En esta clase tenemos opciones para registrar un nuevo
usuario y para añadir conexiones.

Las conexiones (clase `JabberConnection`) por su parte, mantienen los datos del servidor y el usuario y son las encargadas del tráfico
de datos y mensajes.
 
Es decir: `JabberManager` simplemente mantiene una lista de conexiones (propiedad `Connections`) y agrupa los eventos que
vienen de las diferentes conexiones que tengamos abiertas para tratarlos de forma centralizada en nuestras aplicaciones. Además,
dispone de un método llamado `RegisterUser` desde el que podemos dar de alta un usuario en un servidor dado que en ese momento
aún no disponemos de ninguna conexión válida.

## Formularios de la aplicación

Se puede manejar completamente **BauMessenger** desde su formulario principal:

![Formulario principal de BauMessenger](/blog/articles/source-code/baumessenger/formulario-principal.jpg "Formulario principal de BauMessenger")
 
El árbol de la derecha muestra la lista de conexiones y contactos. Podemos registrar un nuevo usuario con el botón de *Crear
usuario* de la barra de herramients superior. Al registrar un nuevo usuario se nos muestra un formulario con los datos necesarios
para crear una cuenta:

![Creación de nueva cuenta XMPP con BauMessenger](/blog/articles/source-code/baumessenger/creacion-usuario.jpg "Creación de nueva cuenta XMPP con BauMessenger")

El formulario que aparece es una de las particularidades del protocolo XMPP: no existe una forma predefinida para crear un usuario si no
que el servidor al que nos conectamos nos envía un XML con los datos que necesita y el cliente los rellena y reenvía para dar de
alta el usuario. Es decir, nos encontraremos con servidores que nos pidan usuario y contraseña simplemente, otros nos pedirán además
un captcha y nuestro nombre. Las posibilidades son ilimitadas.
	
**Nota:** en esta versión no están implementados todos los posibles controles del formulario. Por ejemplo, los
captcha se presentan como URL y no como imagen y no presenta combos. Recordemos que esto comenzó siendo una prueba de concepto.
	
Si ya tenemos un usuario podemos añadir la conexión a nuestra colección con el segundo botón por la derecha de la barra de herramientas
superior:

![Conexión a un servidor XMPP con BauMessenger](/blog/articles/source-code/baumessenger/conexion-a-servidor.jpg "Conexión a un servidor XMPP con BauMessenger")

**Nota:** como dijimos anteriormente, hay varios servidores disponibles. Yo suelo utilizar **jabberes.org** pero
la aplicación debe ser capaz de comunicarse con cualquier servidor.

Una vez tenemos una conexión nos podemos conectar y desconectar así como cambiar nuestro estado con el menú del primer
botón de la barra superior:

![Menú de conexión y desconexión y cambio de estado de BauMessenger](articles/source-code/baumessenger/menu-de-conexion.jpg "Conexión / desconexión")

Por supuesto, podemos añadir contactos una vez conectados a un servidor simplemente añadiendo su nombre de usuario. El nombre de
usuario en XMPP es similar a las cuentas de correo, es decir, *Nombre de usuario@servidor*:

![Añadir un contacto a la lista de contactos de BauMessenger](/blog/articles/source-code/baumessenger/anadir-contacto.jpg "Añadir contacto")

**Xmpp** define que al añadir un contacto a nuestra lista se le envíe un mensaje al usuario destino para que acepte o rechace
nuestra solicitud. Del mismo modo si algún otro usuario solicita añadirnos a su lista de contactos, se abrirá una ventana
indicando que aceptemos o rechacemos su solicitud.
	
**Nota:** para mis pruebas he utilizado un par de clientes estándar, [Pidgin](https://www.pidgin.im/)
y [Gajim](https://gajim.org/index.php?lang=es). Hay muchos más si buscáis por Internet
y no los elegí por ninguna razón en especial, simplemente me pillaban a mano. También hay varios servidores disponibles
como [OpenFire](http://www.igniterealtime.org/projects/openfire/) que podemos utilizar
para probar opciones avanzadas desde nuestro propio ordenador.
	
Por supuesto, lo más importante de una aplicación de mensajería es que podamos enviar y recibir mensajes. Para comenzar una
conversación simplemente debemos pulsar dos veces sobre el icono de un contacto en el árbol de la derecha y se nos abrirá
una ficha de chat con ese usuario:

![Ficha de chat de BauMessenger](/blog/articles/source-code/baumessenger/chat.jpg "Chat de BauMessenger")

Si la comunicación comienza por uno de nuestros contactos, es decir, si otro contacto intenta comunicarse con nosotros, se
abrirá automáticamente una ficha de conversación.

## Resumen

Como vemos, **BauMessenger** es una implementación básica del protocolo XMPP. Permite conectar, crear usuarios, añadir
contactos y enviar y recibir mensajes. Es decir, implementa las opciones principales de una aplicación de mensajería.

Aún así, como decíamos al principio, esto no es más que una aplicación de ejemplo y como tal le faltan opciones importantes:

* No puede abrir ventanas de conferencia o MUC, es decir, no podremos establecer conversaciones con un grupo.
* No envía ni recibe archivos aunque la librería **Sharp.Xmpp** está preparada para ello.
* No tiene opciones avanzadas como videoconferencia aunque el protocolo lo permite.
	
Os invito a ampliar la librería y la aplicación con vuestras ideas y a investigar en todas las posibilidades de este protocolo.