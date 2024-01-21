+++
title = "Archivos de datos"
date = "2019-10-11"
description = "Visualización de archivos de datos CSV o Parquet oon BauDbStudio"
thumbnail = "/applications/baudbstudio/manual/095-archivos-de-datos/095-archivos-de-datos.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

Al trabajar con bases de datos, es muy habitual tener archivos de datos en formato CSV o
[Parquet](https://parquet.apache.org/) o Excel.
	
Para facilitar el trabajo, **BauDbStudio** permite visualizar este tipo de archivos. Simplemente pulsando
dos veces sobre el nodo con el nombre del archivo:

![Arbol de archivos](/blog/applications/baudbstudio/manual/095-archivos-de-datos/arbol-archivos.jpg "Arbol de archivos en BauDbStudio")
			
En la sección central de la aplicación se nos abrirá el contenido del archivo:

![Visualizador de archivos parquet](/blog/applications/baudbstudio/manual/095-archivos-de-datos/visualizador-archivo-parquet.jpg "Visualizador de archivos parquet en BauDbStudio")
			
Si tiene instalado un servidor ODBC de Spark, puede escribir 
[consultas SQL](/blog/applications/baudbstudio/manual/090-consultas/090-consultas) directamente para ver el contenido del archivo.

**Nota:** en los archivos Excel sólo se visualiza la primera hoja sin formato.