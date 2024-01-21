+++
title = "Tutorial de BauDbStudio"
date = "2019-10-11"
description = "Página inicial del tutorial de BauDbStudio"
thumbnail =	"/applications/baudbstudio/manual/001-tutorial-baudbstudio/01-tutorial-baudbstudio.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

Cuando se ejecuta la aplicación **BauDbStudio** por primera vez, nos enfrentamos a un escenario vacío
que debemos configurar antes de comenzar a utilizar la aplicación.
	
Vamos a ver las diferentes partes de la ventana antes de continuar:

![Pantalla inicial](/blog/applications/baudbstudio/manual/001-tutorial-baudbstudio/baudbstudio-primera.jpg "Pantalla inicial de BauDbStudio")
		   
Como cualquier aplicación de Windows, en la parte superior encontramos los menús y las barras de herramientas.

* A la izquierda tenemos una ficha con dos pestañas: archivos y conexiones.
	* En la ficha de [archivos](/blog/applications/baudbstudio/manual/010-archivos/010-archivos) vemos el árbol de nuestra solución 
	y nuestros proyectos una vez seleccionados (en un momento explicamos cómo).
	* En la ficha de [conexiones](/blog/applications/baudbstudio/manual/020-conexiones/020-conexiones) tendremos la lista de conexiones
	a base de datos y [distribuciones](/blog/applications/baudbstudio/manual/025-distribuciones/025-distribuciones).
* La parte derecha de la ventana contiene una ficha donde podemos mantener una lista de conexiones a 
[Azure blob storage](/blog/applications/baudbstudio/manual/030-storage/030-storage) asociadas a nuestros proyectos de datos.
* En la parte central, por ahora vacía, tenemos el espacio donde editaremos y visualizaremos nuestros 
[scripts SQL](/blog/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql), 
[consultas de base de datos](/blog/applications/baudbstudio/manual/090-consultas/090-consultas) o
el contenido de los [archivos de datos](/blog/applications/baudbstudio/manual/095-archivos-de-datos/095-archivos-de-datos) (CSV o parquet).
	
En la parte inferior de la ventana vemos una pestaña de [log](/blog/applications/baudbstudio/manual/100-log/100-log) donde
se muestran los mensajes y errores de la aplicación.
		
Por supuesto, puede cambiar el aspecto de la ventana en cualquier momento y mover las pestañas al punto de la pantalla donde
le resulte más cómodo.

## Configuración inicial

Para comenzar a utilizar la aplicación, le recomiendo que siga los siguientes pasos:

1. Configure las [conexiones a base de datos](/blog/applications/baudbstudio/manual/020-conexiones/020-conexiones) con las que
va a trabajar (si va a utilizar Spark Sql, recomiendo que lea también cómo establecer una
[conexión a Spark con ODBC](/blog/applications/baudbstudio/manual/025-conexion-spark/025-conexion-spark).
2. Seleccione las [carpetas](/blog/applications/baudbstudio/manual/010-archivos/010-archivos) de su ordenador donde 
se van a encontrar los [scripts SQL](/blog/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql),
[scripts de ETL](/blog/applications/baudbstudio/manual/120-scripts-etl/120-scripts-etl) y 
[archivos de datos](/blog/applications/baudbstudio/manual/095-archivos-de-datos/095-archivos-de-datos).
3. Aprenda a ejecutar [consultas](/blog/applications/baudbstudio/manual/090-consultas/090-consultas) sobre las
conexiones de base de datos
4. Aprenda qué son los [archivos de parámetros](/blog/applications/baudbstudio/manual/045-archivos-de-parametros/045-archivos-de-parametros) y
para qué se utilizan.
5. En el caso de trabajar con **Databricks**, prepare las [distribuciones](/blog/applications/baudbstudio/manual/025-distribuciones/025-distribuciones) adecuadas.
6. Si necesita acceder a archivos que se encuentren en **Azure Blob Storage** configure las diferentes
[conexiones a storage](/blog/applications/baudbstudio/manual/030-storage/030-storage). 

## Instalación

Para instalar la aplicación, puede descargar la última versión desde la página de [GitHub](https://github.com/jbautistam/BauDbStudio/releases).
	
La aplicación necesita que tenga instalado en su ordenador la versión 3.0 o superior de
[.Net Core](https://dotnet.microsoft.com/download/dotnet-core/3.1).