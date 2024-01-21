+++
title = "Cómo conectar a Spark"
date = "2019-10-11"
description = "Conexión a Spark desde BauDbStudio"
thumbnail = "/applications/baudbstudio/manual/025-conexion-spark/025-conexion-spark.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

Para poder ejecutar scripts de **Spark SQL** sobre un servidor de Spark, antes tenemos que crear una conexión adecuada.
	
Debemos tener en cuenta que los scripts que creemos sobre la aplicación, se van a ejecutar utilizando ODBC, es decir,
tenemos que tener un servidor ODBC en nuestro equipo o sobre nuestro cluster de Spark.
	
En mi caso, instalé [Spark sobre WSL](/blog/articles/big-data/instalacion-spark/instalacion-spark) en mi equipo y la versión de
64 bits del driver [ODBC para Spark de Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=49883).
	
Una vez instalado Spark, en lugar de levantar los nodos con **start-master** y **start-slave**, debemos levantar
el servidor [Thrift Server] (http://www.russellspitzer.com/2017/05/19/Spark-Sql-Thriftserver/) para Spark ejecutando la siguiente instrucción sobre una consola de WSL:
 
```bash
$SPARK_HOME/sbin/start-thriftserver.sh
```

Al ejecutar esta instrucción, veremos que se levanta el servidor:

![Conectar al servidor Thrift Server](/blog/applications/baudbstudio/manual/025-conexion-spark/servidor-odbc-spark.jpg "Conectar al servidor Thrift Server de Spark para ODBC")
			
Al cabo de unos segundos podemos ver el estado del servidor abriendo un navegador y solicitando la dirección **https://localhost:4040**:
	
![Estado de Thrift Server](/blog/applications/baudbstudio/manual/025-conexion-spark/estado-thrift-server.jpg "Estado del servidor ODBC Thrift Server")
 
En esa página encontraremos información sobre los nodos y el estado de las consultas SQL que ejecutemos sobre el servidor.

Una vez levantado el servidor, simplemente nos queda crear una conexión ODBC utilizando el driver que nos hemos descargado
anteriormente. Para ello, abrimos la ventana de orígenes de datos de 64 bits de Windows:
 	
![Conexiones ODBC de 64bits](/blog/applications/baudbstudio/manual/025-conexion-spark/conexiones-odbc-64.jpg "Conexiones a orígenes de datos ODBC de 64bits de Windows")	
			
Y agregamos una conexión utilizando el driver:

![Creación de una conexion ODBC](/blog/applications/baudbstudio/manual/025-conexion-spark/conexion-odbc-spark.jpg "Creación de una conexion a ODBC Spark")
			
En mi caso, he seleccionado la dirección del host como local (127.0.0.1) en el puerto 10.000 a la base de datos default. Los datos
predeterminados, por cierto:
	
![Conexion de ejemplo](/blog/applications/baudbstudio/manual/025-conexion-spark/conexion-odbc-spark-sample.jpg "Conexion de ejemplo al servidor ODBC de Spark")
			
Como autentificación, he utilizado el mecanismo **User name** y como nombre de usuario el administrador
de mi sistema Linux.

## Configuración en BauDbStudio

Una vez hemos creado la DSN de sistema al servidor Thrift Server de Spark, sólo nos queda crear una 
[conexión](/blog/applications/baudbstudio/manual/020-conexiones/020-conexiones) a nuestro servidor en BauDbStudio.
	
Como el resto de las conexiones, simplemente agregamos una nueva conexión, seleccionamos el tipo de conexión Spark y en
la cadena de conexión escribimos **DSN=LocalSpark** (donde LocalSpark es el nombre de la conexión ODBC que hemos
configurado anteriormente en los orígenes de datos ODBC):
	
![Conexión a un servidor ODBC de Spark](/blog/applications/baudbstudio/manual/025-conexion-spark/conexion-spark-baudbstudio.jpg "Conexión a un servidor ODBC de Spark utilizando BauDbStudio")	
			
**Nota:** como vamos a ejecutar scripts en teoría pesados sobre nuestro servidor a Spark, yo le suelo
dejar un timeout para la ejecución de scripts bastante alto, de hecho, 120 minutos en la imagen.
	
A partir de este momento podemos utilizar el servidor a Spark como cualquier otra base de datos:

![Consulta sobre un servidor Spark](/blog/applications/baudbstudio/manual/025-conexion-spark/consulta-spark.jpg "Consulta sobre un servidor Spark utilizando ODBC en BauDbStudio")