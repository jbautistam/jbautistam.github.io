+++
title = "Visor de archivos PGN I"
date = "2019-10-11"
description = "Primera versión de un visor de archivos de partidas de ajedrez (PGN)"
thumbnail = "/articles/source-code/visor-pgn/visor-pgn.jpg"
tags = [ "Aplicaciones", "Programación" ]
+++

Una de mis deudas pendientes este año, era crear un visor de [archivos PGN](https://es.wikipedia.org/wiki/Notaci%C3%B3n_portable_de_juego) y
hoy ya tengo la primera versión consumible.

En esta quincena, he detenido el proyecto de generación de informes en el que estaba trabajando y me he tomado
unos días libres para escribir un pequeño visor de partidas en WPF. Por supuesto el proyecto era 
ligeramente más ambicioso y he tenido que refrenarme un poco. No quería que se hiciera eterno, aburrirme y abandonarlo
antes de tener una prueba de concepto medianamente interesante. 

Por la presión del tiempo, en lugar de crear mi propio intérprete de archivos PGN decidí utilizar uno ya existente, 
en concreto [PGN.Net](https://github.com/iigorr/pgn.net), escrito a caballo entre F# yC#.
Por esta vez, me he saltado mi orgullo como hice con el 
[intérprete de cron](/blog/articles/opinion/interprete-de-cron/interprete-de-cron)
y he reutilizado un proyecto ya terminado.

Esta librería ya devuelve la partida cargada del archivo PGN con sus movimientos, comentarios y anotaciones, aunque
utilizarlo desde el visor me ha dado algún quebradero de cabeza.

En primer lugar, **PGN.Net** sólo devuelve el texto del movimiento, por ejemplo **c5**, así que para poder
visualizarlo hay que ir a buscar la pieza que puede llegar a esa posición. Esa ha sido la parte más
complicada: la búsqueda de piezas en el tablero y la generación de movimientos de cada una de ellas para comprobar
si pueden llegar a ese escaque. Es decir, internamente tenemos que saber cómo se mueven las piezas y crear un 'minijugador'
de ajedrez por supuesto sin ninguna inteligencia.

El segundo problema se deriva del anterior: **PGN.Net** no crea internamente una reproducción de la partida si no que
simplemente interpreta el texto del archivo. Por eso, en la mayoría de las ocasiones no indica la pieza que se está moviendo a menos
que el texto del movimiento sea explícito (como por ejemplo en #b Nf4#). Lo normal es que nos encontremos una indicación de
movimiento de un peón aunque se haya movido cualquier otra pieza. Una vez salvado el problema anterior, uno de
los ajustes que tiene que hacer el generador es precisamente recuperar la pieza movida y la capturada para no complicarnos en exceso
a la hora de mostrar el movimiento en pantalla.

Una vez interpretado el archivo, la visualización debería ser trivial aunque en mi caso decidí transformar cada movimiento
en acciones para facilitar la representación. Así, tenemos tres tipos de acciones: movimiento, borrado y promoción y 
cada movimiento puede estar compuesto por una o más acciones. Por ejemplo:

* Una captura tiene dos acciones: mover la pieza del jugador y eliminar la pieza comida.
* Una promoción tiene dos acciones: eliminar la pieza del jugador y crear una nueva pieza con la promoción en la
celda de destino, por supuesto, del mismo color que el jugador. Este tipo de movimientos puede llegar incluso
a tener tres acciones si la promoción es resultado de una captura pero supongo que se entiende.
* Un enroque tiene dos acciones: mover la torre y mover el rey.

Así, el visor simplemente tiene que ir reproduciendo las diferentes acciones que componen un movimiento sin preocuparse
realmente del tipo de movimiento que es. Además, que el usuario vaya hacia atrás en los movimientos 
se convierte en algo trivial: para deshacer un movimiento recorremos la colección de acciones hacia
atrás y mostramos el resultado en pantalla.

Como decía al principio, me he obligado a hacer una 'aplicación mínima', no la aplicación que tenía en mente, por eso
he dejado algunas cosas en el tintero:

* No he encontrado ninguna partida en la que hubiese capturas en passant por tanto no he podido comprobar este movimiento.
Tampoco he visto ninguna promoción hasta ahora. *Sí, lo reconozco, he buscado poco y he sido vago. Me podía haber
creado mi propia partida pero...*
* Tampoco he encontrado ninguna partida a medias. En teoría **PGN.Net** interpreta el tablero inicial y lo devuelve en una 
estructura de piezas / escaques pero ahora mismo el visor sólo puede representar partidas completas.
* **PGN.Net** devuelve información con anotaciones que indican si es una buena o mala jugada e incluso el tiempo de cada
una de ellas. Por ahora me he saltado esta parte.
* También incluye notación recursiva de posibles movimientos a partir de un movimiento dado.
Ahora mismo, ni se interpretan ni se visualizan estas bifurcaciones en la partida.
* La aplicación permite ir un movimiento hacia delante o hacia atrás. No podemos seleccionar un movimiento en
la lista y verlo directamente. No debería ser complicado pero me lo he saltado.
* No estoy aprovechando las animaciones de WPF para mover las piezas. Tampoco debería ser complejo: a partir de las 
acciones sólo tenemos que crear un timeline para las diferentes piezas pero sigue en la lista de cosas por hacer. 
No muestra tampoco las celdas por las que va a pasar la pieza que podría ser interesante para el usuario. Tendremos 
que vivir con ello.
* En archivos con más de cien partidas, el intérprete se vuelve algo lento. El problema es que estoy reproduciendo y
buscando los movimiento de las piezas del juego en la carga, no en la visualización. Sería más adecuado almacenar 
los movimientos en la carga sin hacer una búsqueda de las piezas reales e implementar esta lógica de movimientos en 
el visor y no en el generador. Me he dado cuenta a última hora al probar con un archivo con las quinientas mejores 
partidas de Fischer... espero solucionarlo pronto.
* La idea inicial era añadir una base de datos de partidas y permitir la búsqueda de juegos del tipo *aquellos
en que Fischer juegue con blancas y empate* pero se ha quedado en el tintero hasta la próxima versión.

Eso sí, al menos podemos cambiar las imágenes del tablero y las piezas, algo es algo.

La intención para el futuro es solucionar estos problemas y crear una aplicación UWP (sigo sintiéndome más 
cómodo escribiendo este tipo de prototipos en WPF). En cuanto tenga tiempo... por ahora vuelvo al generador de informes, mucho
menos entretenido pero igualmente interesante.

Si alguno quiere echarle un vistazo o proponer modificaciones, podeis encontrar el código fuente en mi cuenta de
[GitHub](https://github.com/jbautistam/BauChessViewer).
