+++
title = "Lector ePub"
date = "2019-10-11"
description = "Código fuente de un lector / generador de libros en formato ePub"
thumbnail = "/articles/source-code/lector-epub/lector-epub.jpg"
tags = [ "Programación" ]
+++

Estos días ando actualizando mis librerías y proyectos a las nuevas versiones de .Net estándar. 
Comienzo a programar con UWP y Xamarin Forms y me veo compilando y recompilando para diferentes versiones
de framework, algo que me está trayendo por el camino de la amargura.
	
Pero también me da algunas sorpresas como reencontrarme con código antiguo que nunca he publicado y
a alguien le podría llegar a interesar. Entre ellas me he encontrado una implementación de un lector / creador
de libros digitales en formato ePub.
	
El [formato ePub](https://es.wikipedia.org/wiki/EPUB), junto con el formato mobi,
quizá sea el tipo de archivo más conocido para la distribución de libros electrónicos y pese a lo que pueda parecer
es un formato bastante sencillo.
	
En primer lugar, los archivos ePub no son más que archivos comprimidos en formato zip, por lo que si les cambiamos
la extensión los podemos descomprimir sin ningún problema.
	
En el directorio donde hayamos descomprimido el zip nos vamos a encontrar un archivo y varios subdirectorios:

* El archivo `mimetype` indica el tipo de eBook. En mi libro de ejemplo, la única línea que contiene es `application/epub+zip`.
* El directorio `META-INF` contiene el archivo `container.xml` que indica la dirección del archivo
donde se encuentra el contenido del libro digital y los archivos adicionales como índices. En mi 
ejemplo, indica que este archivo está en `OEBPS/content.opf`.
* El directorio `OEBPS`, en mi caso, contiene los archivos de texto de las páginas del libro, normalmente
en formato html junto con las imágenes y los estilos en uno o varios directorios. En este mismo directorio
hay un archivo especial llamado `content.opf` que sirve como índice del libro indicando las páginas
y el orden.
		
Así la librería de interpretación se convierte en algo sencillo, simplemente descomprime el archivo ePub en un directorio,
busca e intepreta los archivos de índice y expone esta información en una colección de páginas.
	
La aplicación (Windows Forms en esta ocasión), sólo tiene que recorrer las páginas y mostrarlas en un árbol y abrir los
archivos HTML que el usuario haya seleccionado.
	
Lo único especial es que dado que hay diferentes formatos de libros digitales, la librería lee el ePub en una estructura,
que en teoría, debería ser neutra para todos los formatos: una colección de páginas / archivos y un árbol de nodos índice.
Este formato neutro lo podríamos de hecho utilizar para convertir libros a diferentes formatos o para presentarlos sin
tener que preocuparnos del tipo de eBook que hemos leído.
	
La librería también está preparada para crear libros en formato ePub pero actualmente no tenemos forma de hacerlo desde la 
interface de usuario. Tampoco puede leer nada que no sea formato ePub. Dos mejoras que dejo como ejercicio al lector
o para próximas versiones.
	
Por último y no menos importante, podéis ver / descargar este proyecto desde 
[GitHub](https://github.com/jbautistam/ePubLibrary).

## Distribución de los proyectos

En la solución hay ocho proyectos: 
 
* `BauControls` y `WebExplorer` son controles WindowsForms y se utilizan sobre todo como controles de selección de archivos y para
la visualización de las páginas HTML.
* `LibHelper.Standard`, `LibMarkup.Standard`, `LibCompressor` y `LibSystem` son proyectos de apoyo con
funciones comunes y extensores, lectura de XML, compresión / decompresión de archivos y funciones de sistema
como API y manejo de archivos respectivamente.
* `LibEBooks` es la librería que interpreta y crea los archivos ePub.
* `ePub` es la aplicación Windows Forms que lee y muestra los archivos apoyándose en la librería anterior.

## Problemas molestos de.Net Standard

Como decía al principio, estoy pasando parte de mis librerías a Net estándar y me estoy encontrando con algunos problemas.
Posiblemente algo puede que esté haciendo mal pero también parece que es ligeramente inestable.
	
En primer lugar, la aplicación principal, los controles de Windows Forms y algunas librerías (como la de ePub) 
se compilan utilizando el framework 4.7. Por su parte, las librerías de compresión, lectura de XML y funciones
comunes (extensores sobre todo) utlizan Net Standard.
	
La librería de compresión, utiliza un paquete de NuGet (`SharpCompress`, os lo recomiendo). Sin embargo, cuando
la llamamos desde la librería de lectura de ePubs, nos da un error indicando que no se puede cargar `SharpCompress`.
La solución ha pasado por añadir `SharpCompress` a la librería de ePubs aunque esta librería no acceda nunca a ella
dado que es la de compresión quien hace este trabajo.
	
Lo más curioso es que el error no da en tiempo de compilación si no de ejecución.
	
Otra curiosidad de esta librería de compresión: está compilada en Net Standard 1.4 que no define `System.IO`. Tiene
que crear directorios a la hora de descomprimir archivos y por tanto tenemos un problema. Lo he solucionado lanzando
eventos que recoge LibEPub en este caso para crear los directorios y borrar los archivos antiguos: feo cuanto menos.
En teoría Net Standard 1.6 puede utilizar `System.IO`, seguid leyendo y veréis porqué no utilizo esta versión.

Algo incluso peor: todas las librerías que utilizan Net Standard en este proyecto, usan la versión 1.4 excepto la de lectura
de XML que da un error similar en ejecución con la carga de `System.Xml.Reader`. Seguí el mismo patrón que en el caso
anterior: instalar el paquete NuGet de `System.Xml.Reader` sobre la librería de ePub aunque no lo utilice directamente, pero
me dio otro error de carga, esta vez de `System.Runtime` que me pareció mucho más raro.
	
La solución pasó por bajar la librería de interpretación de XML a la versión 1.2 y funcionó correctamente. No digo que sea
la única solución pero fue la primera que se me ocurrió.
	
Así que en esta solución hay librerías con Net Standard 1.4 y 1.2 y he tenido que añadir paquetes NuGet en lugares que a 
mi modo de ver no eran necesarios y de la peor forma posible, guiado por errores en tiempo de ejecución y no de compilación.
	
Ahora que estamos a las puertas de la versión 2.0, parecería lógico utilizar la versión de Net Standard 1.6 pero en realidad
no se admite con ningún framework de.Net, ni siquiera el 4.7 (por cierto, yo me lo descargué precisamente para eso aunque
vendrá bien para comenzar a  utilizarC# 7.0):

![Relación de Net Standard con los frameworks de .Net](/blog/articles/source-code/lector-epub/netstandard.jpg "Net Standard")
			
No deja de sorprenderme, no es por criticar a Microsoft (bueno, sí), pero el cambio Net.Core, Net.Standard, Net.Framework
que parecía cosa de unos meses está dando lugar a un ecosistema bastante "inestable" cuanto menos y ahora mismo una fuente
de incertidumbre considerable.
	
Veremos qué pasa cuando se lance Net.Standard 2.0.