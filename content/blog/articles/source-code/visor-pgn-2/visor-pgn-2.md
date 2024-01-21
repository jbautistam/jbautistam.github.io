+++
title = "Visor de archivos PGN II"
date = "2019-10-11"
description = "Segunda versión del visor de archivo PGN"
thumbnail = "/articles/source-code/visor-pgn-2/visor-pgn-2.jpg"
tags = [ "Aplicaciones", "Programación" ]
+++

Hace un par de semanas publiqué la primera versión de un pequeño 
[visor de archivos PGN](/blog/articles/source-code/visor-pgn/visor-pgn).
	
En esa primera versión dejé una lista de cosas pendientes que llevan molestándome desde entonces, así que
en tiempos muertos he desarrollado algunas modificaciones que mejoran (creo) la aplicación.
	
Esta segunda versión, aparte de corregir algunos errores, trae las siguiente modificaciones:

* Ha mejorado la velocidad en la lectura de partidas: ahora cargar las quinientas mejores partidas de Bobby Fischer es
algo más llevadero.
* He incorporado la lectura de partidas a medias y me ha resultado muy útil para los archivos de problemas de tipo 'blancas
ganan en tres movimientos'.
* Ya se pueden seleccionar movimientos en la lista y verlos directamente sin tener que ir uno por uno.
* En la lista de movimientos se muestran las variaciones.
* He añadido la visualización de las etiquetas de partida en las que se recoge por ejemplo el ELO de los jugadores o
los tipos de apertura.

Pero siguen quedando algunas cosas pendientes:

* Sigo sin probar las capturas en passant. 
* No se aprovechan las animaciones de WPF para los movimientos.
* La posibilidad de consultar una base de datos de partidas creo que va a tener que esperar algún tiempo más.
* Aunque ya se muestran las variaciones en la lista de movimientos, por ahora no podemos ejecutarlos en el tablero.
* No podemos escribir nuestros propios archivos PGN. Eso me obligaría a permitir movimientos manuales y posiblemente
cambiase el modelo de vistas. No sé si esto acabará convirtiéndose en un requisito pero por ahora está muy abajo
en la lista.
* La versión UWP va a sufrir un retraso indeterminado.

Por supuesto, el código fuente sigue estando en: [GitHub](https://github.com/jbautistam/BauChessViewer).