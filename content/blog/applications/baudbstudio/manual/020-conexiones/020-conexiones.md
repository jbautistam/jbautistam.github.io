+++
title = "Conexiones a base de datos"
date = "2019-10-11"
description = "Configuración de conexiones a base de datos en BauDbStudio"
thumbnail = "/applications/baudbstudio/manual/020-conexiones/020-conexiones.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

Para poder trabajar con base de datos en **BauDbStudio** primero debemos crear las conexiones oportunas.

Inicialmente, las conexiones definidas a base de datos se ven en el panel de la izquierda en la ficha **Conexiones**:

![Ficha conexiones a base de datos](/blog/applications/baudbstudio/manual/020-conexiones/ficha-conexiones.jpg "Ficha conexiones a base de datos en BauDbStudio")

Para crear una conexión, lo podemos hacer tanto desde el menú principal con la opción *Archivo | Nuevo | Conexión*
como con el icono de la barra de herramientas como desde el menú secundario sobre el nodo de conexiones:

![Crear una nueva conexión](/blog/applications/baudbstudio/manual/020-conexiones/crear-conexion.jpg "Crear una nueva conexión a base de datos en BauDbStudio")

Con cualquiera de estas opciones, se nos abre el cuadro de diálogo de creación de una nueva conexión:

![Crear conexión](/blog/applications/baudbstudio/manual/020-conexiones/cuadro-dialogo-conexion.jpg "Crear una nueva conexión en BauDbStudio")

En **BauDbStudio** se pueden crear conexiones diferentes servidores de base de datos:

* Spark
* Sql Server
* SqLite
* MySql
* PostgreSql
* ODBC
	
**Nota:** la conexión a Spark es algo especial, puede encontrar más información en [Crear una conexión Spark](/blog/applications/baudbstudio/manual/025-conexion-spark/025-conexion-spark).
	
Para cada una de ellas, por supuesto, tendremos que rellenar los datos adecuados, por ejemplo, para una conexión SQL Server
debemos indicar el servidor, puerto, base de datos y usuario y contraseña:
	
![Creación de una conexión](/blog/applications/baudbstudio/manual/020-conexiones/conexion-sql-server.jpg "Creación de una conexión a un servidor Sql Server con BauDbStudio")

Una vez creada la conexión, al pulsar dos veces sobre su nombre en el árbol o al seleccionar 
	la opción **Abrir** en el menú que aparece cuando pulsamos con el botón secundario del ratón sobre un nodo de conexión,
	podemos modificar los datos de la conexión:
	
![Modificar una conexión](/blog/applications/baudbstudio/manual/020-conexiones/modificar-conexion.jpg "Modificar una conexión a base de datos en BauDbStudio")

Una vez creada la conexión, nos aparecerá en el árbol de conexiones y podemos ver el esquema de datos: tablas y campos:

![Esquema de base datos](/blog/applications/baudbstudio/manual/020-conexiones/esquema-base-datos.jpg "Esquema de base datos de una conexión en BauDbStudio")

Al pulsar dos veces sobre una tabla nos aparecerá una ventana de [consulta](/blog/applications/baudbstudio/manual/090-consultas/090-consultas):

![Consulta de los datos de una tabla](/blog/applications/baudbstudio/manual/020-conexiones/consulta-tabla.jpg "Consulta de los datos de una tabla en BauDbStudio")
			
donde podremos ver el contenido de la tabla.
