+++
title = "Directorios de trabajo"
date = "2019-10-11"
description = "Tratamiento de los directorios de trabajo en BauDbStudio"
thumbnail = "/applications/baudbstudio/manual/010-archivos/040-scripts.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

Para trabajar con archivos de script de SQL o ETL u otros archivos, primero debemos asociar directorios
al proyecto en BauDbStudio.
	
Para añadir un directorio al []({ href = "Aplicaciones\BauDbStudio\Manual\110 Marco trabajo" } marco de trabajo)
actual de la aplicación podemos utilizar tanto la opción del menú **Archivo | Nuevo | Directorio** como el 
botón **Añadir carpeta** de la barra de herramientas como el menú secundario de la ficha **Archivos**:

![Añadir un directorio al árbol de archivos](/applications/baudbstudio/manual/010-archivos/nuevo-directorio.jpg "Añadir un directorio al árbol de archivos")
			
Sólo debemos seleccionar un directorio de nuestro sistema donde vamos a almacenar nuestros archivos SQL o archivos de
datos.
	
Para los ejemplo de este tutorial, me he creado éste árbol de directorios:

![Arbol de directorios de ejemplo](/applications/baudbstudio/manual/010-archivos/arbol-directorios.jpg "Arbol de directorios de ejemplo")
			
Sobre el árbol de directorios podemos crear carpetas o archivos de diferente tipo 
[scripts de SQL](/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql),
[scripts para procesos ETL](/applications/baudbstudio/manual/120-scripts-etl/120-scripts-etl) o 
[archivos de datos](/applications/baudbstudio/manual/095-archivos-de-datos/095-archivos-de-datos)
CSV o parquet.
	
Normalmente suelo dejar por separado los directorios donde almaceno archivos de datos de los scripts porque así
puedo asociar las carpetas de proyecto a un repositorio de control de código fuente pero en este caso de ejemplo
he dejado todos los archivos sobre la misma carpeta de trabajo.
	
Por supuesto, al pulsar dos veces sobre cada uno de los archivos se nos abrirá el editor o visualizador adecuado:

![Editor de SQL de BauDbStudio](/applications/baudbstudio/manual/010-archivos/visualizador-sql.jpg "Editor de SQL de BauDbStudio")