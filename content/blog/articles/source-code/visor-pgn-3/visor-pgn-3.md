+++
title = "Visor de archivos PGN III"
date = "2019-10-11"
description = "Programa para visualización de partidas de ajedrez en archivos PGN"
thumbnail = "/articles/source-code/visor-pgn-3/visor-pgn-3.jpg"
tags = [ "Aplicaciones", "Programación" ]
+++

Espero que nadie estuviera aguantando la respiración desde que reescribí la aplicación 
[BauChessViewer](/blog/articles/source-code/visor-pgn-2/visor-pgn-2) hace un par de años ya, pero
hasta estos meses no he encontrado el tiempo para continuar con ella.

Ha habido otros proyectos entre medias y realmente no pensaba recuperar esta aplicación pero
comencé a estudiar **Xamarin Forms** y, después de terminar con la versión de
[DevConference para Android](Aplicaciones\DevConference Android), creía que
era un buen momento para aplicar los conocimientos adquiridos en un visor de juegos para tablets.

Y el primer problema fue precisamente la librería que utilizaba para leer los archivos PGN,
[PGN.Net](https://github.com/iigorr/pgn.net). 

Como [comentaba](/blog/articles/source-code/visor-pgn/visor-pgn), al comenzar el desarrollo de la aplicación no
me apetecía dedicarle demasiado tiempo, era más una prueba de concepto y, algo no habitual en mí, busqué una librería
para interpretar los archivos en lugar de escribir la mía.

Aunque esta forma de trabajar tiene sus ventajas, cuando fui a portar la aplicación a Xamarin, me encontré con el
primer problema: **PGN.Net** es una librería escrita en F# para el framework 4.5 y por tanto no la podía utilizar
para programar en Xamarin. Podría haber descargado el código y portarlo pero depende de otra librería también escrita
en F# y no me siento cómodo con ese lenguage, así que el siguiente paso lógico era escribir mi propio intérprete de
archivos PGN.

No sé si habéis visto alguna vez un archivo PGN, es algo más o menos así:

```csharp
[Event "London Chess Classic"]
[Site "London"]
[Date "2009.12.13"]
[Round "5"]
[White "Ni, Hua"]
[Black "Carlsen, Magnus"]
[Result "0-1"]
[WhiteElo "2665"]
[BlackElo "2801"]
[ECO "B51"]

1. e4 c5 2. Nf3 d6 3. Bb5+ Nd7 4. d4 a6 5. Bxd7+ Bxd7 6. dxc5 dxc5 7. Nc3
e6 8. Bf4 Ne7 9. Ne5 Ng6 10. Qh5 Bc6 11. Bg3 Nxe5 12. Bxe5 c4 13. O-O Qa5
14. Qg5 h6 15. Qg3 f6 16. Qg6+ Ke7 17. Bf4 Be8 18. Qg3 Kf7 19. Rad1 Bc6 20.
Rd2 e5 21. Be3 Bb4 22. f4 Rhe8 23. f5 Bc5 24. Rfd1 Rad8 25. Rxd8 Bxe3+ 26.
Qxe3 Rxd8 27. Rxd8 Qxd8 28. Kf2 Qd6 29. a3 a5 30. Kf3 Kg8 31. g3 b5 32. Ke2
b4 33. axb4 axb4 34. Nd1 Ba4 35. b3 cxb3 36. cxb3 Qa6+ 37. Kd2 Bb5 38. Qc5
Qa2+ 39. Qc2 Qa7 40. Qc8+ Kh7 41. Kc1 Qa1+ 42. Kc2 Qd4 0-1
```

Cómo podéis imaginar, no es un formato pensado para reproducirlo con un ordenador. Realmente es
una adaptación del formato que se utiliza al escribir partidas en libros o periódicos. Un humano debería poderlo leer fácilmente,
un ordenador, no tanto. Y la cosa se complica si además le introducimos comentarios o variaciones.

Así que el primer paso fue precisamente la interpretación y aunque pueda parecer lo contrario, resulta que se adapta bastante
bien a las gramáticas BNF que utilizamos para escribir intérpretes y compiladores:

* Una serie de etiquetas escritas entre corchetes
* Una lista de movimientos compuestas por:
	* Número de movimiento
	* Movimiento en formato SAN (*e4, c5, Bg5+, 0-1*, por ejemplo)

Además, los archivos PGN pueden contener comentarios (entre llaves) y variaciones (entre paréntesis) que hacen que todo sea
mucho más interesante.

La mayoría de los archivos PGN además contienen varias partidas. La separación entre ellos es precisamente
el comienzo de un nuevo bloque de cabeceras.

Por último, una de las cabeceras puede ser el estado inicial del tablero para, por ejemplo, los archivos de tipo *blancas
mueven y ganan en 5* en notación FEN, algo más o menos como esto:

```csharp
[Event "White mates in two moves"]
[Result "1-0"]
[SetUp "1"]
[FEN "1N5r/7b/8/8/2ppp2p/B4Q1r/k3PR1P/1RK5 w - - 0 1"]

1. Bb4 1-0
```	

Aparte de la relativa dificultad de la interpretación del archivo, me resulta muy curioso el formato. Te transporta a tiempos
en que todo tenía que ser más compacto y cuidado. Fijaos en la 
[notación FEN](http://www.edicionesma40.com/blog/la-notacion-fen.htm), describe la posición 
de las piezas en el tablero:

```csharp
1N5r/7b/8/8/2ppp2p/B4Q1r/k3PR1P/1RK5 w - - 0 1
```

Un tablero tiene 64 escaques, cada uno puede tener una pieza blanca o negra que se representa con el código de pieza (un carácter)
en mayúscula si es blanca o en minúsculas si es negra. Yo me habría liado la manta a la cabeza y lo habría representado
con una cadena de 64 caracteres con puntos para las celdas vacías. Sin embargo, la notación utiliza esto: `/8/` para indicar
que esa fila tiene 8 celdas en blanco. Ha ahorrado 5 caracteres. Puede parecer una optimización minúscula, a mí me
parece curiosísimo.

Aparte de eso, la segunda dificultad de estos archivos, es que realmente necesitan un poco de inteligencia, no es sólo saber
interpretarlo si no además, saber cómo se mueven las piezas.

Así un movimiento como **c4** indica que una pieza se mueve a la celda **C4** pero ¿qué pieza? hay que buscar por el tablero
cuál puede ejecutar ese movimiento. Muy divertido. Podríamos pensar en un formato más sencillo con celda
de origen, celda de destino pero sería mucho menos compacto y sobre todo mucho más difícil de leer para un humano.

Tras escribir el intérprete, el siguiente problema fue el propio motor del juego. Sabía demasiado de PGN para mi gusto,
de hecho, sabía más de PGN que del juego. Un desastre. Me impedía hacer cosas como ampiar el motor para leer otros
tipos de archivos o simplemente añadirle funciones para jugar contra un humano. Por tanto, el segundo paso fue separar 
el motor de las librerías de interpretación, para ello creé una librería únicamente con el motor y otra librería de 
conversión. Podría haber utilizado la librería de interpretación como librería de conversión pero tenía el mismo problema
de acoplamiento por lo que decidí separarlo en tres.

Lo bueno de tenerlo separado es que si algún día me decido por fin a crear una base de datos de partidas, lo único que necesito
es la librería de interpretación sin motor ni librería de conversión. Todo eran ventajas.

Y terminado todo esto podía pasar a la aplicación WPF del visor y como suponía, también tuvo su miga:

* Las vistas, sobre todo la del tablero, estaban enlazadas al motor, al cambiar el motor hubo que cambiar tanto los
ViewModel como las propias vistas.
* Los movimientos se hacían en las vistas. Esto no supondría ningún problema si mi propósito no fuera migrar a UWP o
Xamarin Form, pero como el propósito final era precisamente ese, había que separar la lógica de movimientos y llevarlas
al ViewModel.
* Todas las vistas estaban concebidas para hacer un visor de archivos. No es necesariamente malo pero si algún día quería hacer
un motor de juego*em inteligente) me impedía hacer ningún movimiento que no estuviera en un archivo PGN.
* Por si fuera poco, había un error en el concepto de variaciones de una jugada. Suponía que una jugada sólo podría tener
una variación pero no es cierto, una jugada puede tener diferentes variaciones. 

Total, tuve que cambiar toda la lógica tanto de las vistas como de los ViewModel para conseguir que las vistas no supieran
prácticamente nada de lo que estaba pasando, de hecho, incluso los escaques están en el ViewModel y las vistas simplemente los
pintan. Ahora puede que no tenga demasiado sentido, pero si en el futuro próximo incluyo cosas como una visualización de
movimientos posibles de una pieza, puede ir al ViewModel sin modificaciones sobre la vista.

Por último, un resumen de las cosas que he aprovechado para arreglar en esta versión:

* Por fin se pueden leer y mostrar archivos con capturas al paso (*passant*). Parece mentira que los más complicado
del ajedrez sean precisamente los movimientos de peón.
* Ya se pueden ejecutar las diferentes variaciones de piezas. He perdido eso sí la visualización de las variaciones en la
lista. Nunca me llegó a convencer demasiado en cualquier caso.
* Añadí la animación del movimiento de las piezas. Lo hace todo mucho más fluido.

Y para el futuro dejamos:

* Para las personas que aprenden a jugar, no estaría mal que al seleccionar una pieza se mostrasen todos los movimientos
posibles de la mismas.
* La aplicación no hace ningún tipo de análisis. Para la siguiente versión posiblemente incluya visualizaciones de
piezas atacadas o dominio del tablero, por ejemplo.
* Ya podemos leer archivos pero no hay forma de crear los archivos porque no hay forma aún de mover las piezas. Al
fin y al cabo sigue siendo un visor de archivos. ¿Se podría hacer un juego real de ajedrez sobre esta base?
* No estaría mal que el usuario pudiese crear sus propios archivos con las partidas que le resulten interesantes. Lo 
malo de utilizar archivos PGN fijos es que no hay una manera sencilla de encontrar los favoritos a menos que exista
una base de datos asociada.

En resumen, el código UWP / Xamarin Forms y la base de datos, tendrán nuevamente que esperar pero al menos
las bases están mejor desarrolladas y el código y el interface de usuario parece más razonable y debería ser más fácil 
de retomar.

Como siempre, por si alguien quiere echarle un vistazo, el código fuente está en 
[GitHub](https://github.com/jbautistam/BauChess).