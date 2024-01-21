+++
title = "DevConference: conferencias de desarrollo"
date = "2019-10-11"
description = "DevConference: visor para conferencias de desarrollo y programación"
thumbnail = "/applications/devconference/devconference.jpg"
tags = [ "Aplicaciones" ]
+++

Ultimamente, casi a diario tenemos nuevas conferencias y charlas sobre programación o desarrollo. Es prácticamente
imposible asistir a todas aunque, afortunadamente, la mayoría de ellas se graban y se vuelcan en 
plataformas como YouTube o Channel9.

Desgraciadamente, nos las acabamos perdiendo, al menos yo. No se puede seguir la pista a tanta información. Nos enteramos
tarde de las fechas, nos olvidamos o no tenemos tiempo para verlas. Acabamos faltando a las más interesantes
hasta que alguien nos habla sobre ellas y es imposible acordarnos del título de las que queremos volver a ver.

**DevConference** surge con la idea de  solventar estos problemas o al menos facilitar el seguimiento
y la visualización de conferencias y charlas.

**DevConference** se basa en los sistemas de RSS y Podcast: un archivo de agregación que se guarda en un servidor
HTTP y que cualquier usuario puede descargar y mantener en su dispositivo local borrando aquellas que no le 
resulten interesantes, manteniendo sus favoritos o asociándose a nuevos canales.

En esta ocasión, me he decidido por una aplicación UWP. La podemos descargar del 
[Windows Store](ms-windows-store://pdp/?productid=9n628w6kxjjw) o en este 
[enlace](https://www.microsoft.com/es-es/store/p/devconference/9n628w6kxjjw).

## Funcionamiento de la aplicación

Una vez descargada la aplicación de la tienda, se abre la pantalla principal:

![Pantalla principal de la aplicación de DevConference](/blog/applications/devconference/pantalla-principal.jpg "Pantalla principal")
		
Al instalarse por primera vez, descarga los archivos de los canales que mantengo en mi sistema. El usuario puede en cualquier 
momento añadir o modificar sus canales utilizando los iconos de la barra de herramientas superior:

![Barra de herramientas principal de DevConference](/blog/applications/devconference/barra-herramientas-principal.jpg "Barra de herramientas principal")

Para añadir o modificar un canal, basta con darle un nombre y seleccionar la dirección donde se encuentra el
archivo del canal (veremos cómo se crea un archivo de canal más adelante en este mismo artículo):

![Mantenimiento de canales en DevConference](/blog/applications/devconference/creacion-de-canales.jpg "Mantenimiento de canales")

Cuando se descargue este archivo, en el árbol de la izquierda se muestra la relación de canales y categorías donde se indica
el número de archivos pendientes de visualizar:

![Arbol de canales y categorías en DevConference](/blog/applications/devconference/arbol-canales.jpg "Arbol de canales y categorías")
		
Bajo ese árbol de canales, hay otra barra de herramientas con el filtro de entradas (leidas, no leidas o
favoritas) o para cambiar el modo de visualización:

![Barra de herramientas secundaria de DevConference](/blog/applications/devconference/barra-herramientas-secundaria.jpg "Barra de herramientas secundaria")

Para ver un vídeo, basta con pulsar sobre una de las entradas de una categoría:

![Ver vídeo de una conferencia en DevConference](/blog/applications/devconference/ver-video.jpg "Ver vídeo de una conferencia")

## Los formatos de archivos de DevConference

El formato de los archivos de **DevConference** es muy similar a RSS, Atom y OPML aunque más reducidos. 

El archivo más importante, el que los usuarios utilizan al añadir un canal y desde el que se descargan las entradas,
tiene este formato que identifica las categorías y entradas del canal:

```XML
<?xml version='1.0' encoding='utf-8'?>
<DevConference>
	<Category Id = "af39430" >
		<Name>Agile</Name>
		<Conference Id = "4b426ea"  CreatedAt = "2017-11-05 00:48:45"  PublishedAt = "2017-11-05 00:48:45" >
		  <Name>The S.O.L.I.D. Principles of OO and Agile Design - by Uncle Bob Martin</Name>
		  <Summary>UB doesnt talk about -all- 5 SOLID...</Summary>
		  <Authors>Bob Martin</Authors>
		  <Video>https://youtube.com/watch?v=t86v3N4OshQ</Video>
		</Conference>
	</Category>
	<Category Id = "aee42ef" >
		<Name>Arquitectura</Name>
		<Conference Id = "105dd9c" CreatedAt = "2017-11-05 00:51:06" PublishedAt = "2017-11-05 00:51:06" >
		  <Name>Robert C Martin - Clean Architecture and Design</Name>
		  <Summary/>
		  <Authors>Robert C Martin</Authors>
		  <Video>https://youtube.com/watch?v=Nsjsiz2A9mg</Video>
		</Conference>
		<Conference Id = "019659" CreatedAt = "2017-11-05 00:51:50" PublishedAt = "2017-11-05 00:51:50" >
		  <Name>Robert C Martin - The Single Responsibility Principle</Name>
		  <Summary/>
		  <Authors>Robert C Martin</Authors>
		  <Video>https://youtube.com/watch?v=Gt0M_OHKhQE</Video>
		</Conference>
		<Conference Id = "4d88059" CreatedAt = "2017-11-05 00:47:57" PublishedAt = "2017-11-05 00:47:57" >
		  <Name>The Principles of Clean Architecture by Uncle Bob Martin</Name>
		  <Summary>The Principles of Clean Architecture by Uncle Bob Martin</Summary>
		  <Authors>Bob Martin</Authors>
		  <Video>https://youtube.com/watch?v=o_TH-Y78tt4</Video>
		</Conference>
	</Category>
</DevConference>
```

Es este archivo el que debemos mantener en nuestro servidor HTML para distribuir nuestra propia información de 
conferencias o charlas.

El archivo de intercambio utilizado para exportar e importar canales es el siguiente:

```XML
<?xml version='1.0' encoding='utf-8'?>
<Tracks>
	<Track>
		<Name>.Net </Name>
		<Url>http://jbautistam.com/DevConference/NetConferences.xml</Url>
	</Track>
	<Track>
		<Name>Agilidad, arquitectura... </Name>
		<Url>http://jbautistam.com/DevConference/Conferences.xml</Url>
	</Track>
</Tracks>
```

donde se identifican los nombres de los canales y las direcciones desde las que se descargan las entradas (el primer archivo XML
descrito).

## El futuro de la aplicación

La intención inicial de esta aplicación, era aprender lo suficiente de UWP como para lanzar una herramienta medianamente
útil en Windows Store.

Como siempre, el futuro de la aplicación, depende de su acogida y del tiempo que pueda dedicarle estos meses entre
mis otros proyectos. La intención es escribir una aplicación Xamarin Forms que la complemente para así
aprender un poco más de la tecnología. 

Por supuesto, pretendo mantener actualizados mis canales aunque sea sólo para mi propio uso.
