+++
title = "Scripts"
date = "2019-10-11"
description = "Escribiendo scripts de base de datos en BauDbStudio"
thumbnail = "/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

La principal función de **BauDbStudio** es la ejecución de scripts de SQL.

Para crear un script de SQL simplemente debemos añadir un [archivo](/blog/applications/baudbstudio/manual/010-archivos/010-archivos)
con extensión SQL a nuestro directorio de proyecto.
	
Los scripts a ejecutar en **BauDbStudio** se componen de instrucciones SQL (SELECT, INSERT, CREATE TABLE...) adecuadas
para la [conexión a base de datos](/blog/applications/baudbstudio/manual/020-conexiones/020-conexiones) que estemos utilizando.
	
El único requisito es que debemos separar las sentencias SQL con el comando **GO** que sirve como separador de sentencias
en **BauDbStudio**.
	
Por ejemplo, veamos un script de creación de una tabla a partir de un archivo parquet utilizando una conexión SparkSql:

```SQL
DROP TABLE IF EXISTS Sales
GO

CREATE TABLE Sales AS
	SELECT *
		FROM parquet.`/mnt/c/Test/BauDbStudio/Samples/Files/Input/Sales.parquet`
GO
```

**Nota:** en mi caso, el archivo parquet se encuentra en el directorio *c:/Test/BauDbStudio/Samples/Files/Input/Sales.parquet*, dado que estoy utilizando Spark con WSL, 
el directorio se debe indicar en formato Linux, por eso he utilizado el nombre de archivo */mnt/c/Test/BauDbStudio/Samples/Files/Input/Sales.parquet*.
	
Si queremos ejecutar este script, debemos seleccionar la 
[conexión a nuestro servidor de Spark](/blog/applications/baudbstudio/manual/025-conexion-spark/025-conexion-spark)
en la barra de herramientas y pulsar el botón de **Ejecutar** de la barra de herramientas:

![Ejecutar script SQL](/blog/applications/baudbstudio/manual/040-scripts-sql/ejecutar-script-sql.jpg "Ejecutar script SQL desde la barra de herramientas")
			
También podemos utilizar la tecla de función **F5** para ejecutar el script. Podemos seleccionar parte del texto 
para ejecutar únicamente ua parte del script.
	
**Nota:** al ejecutar un script, se muestra en la [ventana de log](/blog/applications/baudbstudio/manual/100-log/100-log) el resultado de la ejecución. Si deseamos
ver el resultado de una consulta debemos utilizar la ventana de [consultas](/blog/applications/baudbstudio/manual/090-consultas/090-consultas).
	
Mientras ejecutemos el script, en la barra de herramientas vemos el tiempo de ejecución:

![Ejecución de un script SQL](/blog/applications/baudbstudio/manual/040-scripts-sql/ejecucion-script.jpg "Ejecución de un script SQL")
		   
En cualquier momento podemos detener la ejecución del script pulsando el botón **Cancelar** de la barra de herramientas.

Por supuesto, podemos cerrar la ventana del script y seguir trabajando mientras termina la ejecución.

El script anterior creaba una tabla de ventas con los datos del archivo parquet, una vez ejecutado podemos ver la
tabla creada en nuestra lista de conexiones:

![Creación de una tabla](/blog/applications/baudbstudio/manual/040-scripts-sql/nueva-tabla.jpg "Creación de una tabla en la base de datos")

## Parámetros de consulta

Una vez tenemos la tabla de ventas en nuestra base de datos, siempre podemos crear una tabla de resumen con la suma de datos entre
fechas, por ejemplo:
	
```SQL
DROP TABLE IF EXISTS SalesSummary
GO

CREATE TABLE SalesSummary AS
	SELECT StoreId, ProductId, SUM(Units) AS Units
		FROM Sales
		WHERE Date >= '2017-01-01'
		GROUP BY StoreId, ProductId
GO
```

Este script nos va a dar un acumulado de ventas desde el año 2017, pero ¿no estaría bien que pudiéramos modificar esta fecha
sin tener que volver a escribir el script?
	
Para ello vamos a utilizar argumentos en las consultas y archivos de parámetros.

Para utilizar un argumento en una consulta sobre Spark Sql, vamos a usar un script de este tipo:

```SQL
DROP TABLE IF EXISTS SalesSummary
GO

CREATE TABLE SalesSummary AS
	SELECT StoreId, ProductId, SUM(Units) AS Units
		FROM Sales
		WHERE Date >= $StartDate
		GROUP BY StoreId, ProductId
GO
```

Donde **$StartDate** es nuestro parámetro o la variable que vamos a utilizar para indicar la fecha, es decir, identificamos
los argumentos en Spark Sql con un signo de dólar seguido por el nombre del argumento.
	
Pero antes de ejecutar esta instrucción, debemos indicarle a la aplicación el valor de los parámetros que vamos a utilizar.

Para ello primero creamos un archivo de parámetros en nuestro árbol de archivo y lo llamamos, por ejemplo, *Parameters.json*:

![Creación de un archivo de parámetros](/blog/applications/baudbstudio/manual/040-scripts-sql/creacion-archivo-parametros.jpg "Creación de un archivo de parámetros")
		   
Los archivos de parámetros tienen formato JSON, en nuestro caso, va a tener un único parámetro, nuestra variable **StartDate**,
escribiendo el siguiente texto en el archivo:
	
```JSON
{
	"StartDate": "2017-01-01"
}	
```

Lo último que nos falta es asociar este archivo de parámetros con la ejecución de nuestros scripts. Para ésto, pulsamos el
botón **Seleccionar archivo de parámetros** situado en la barra de herramientas a la derecha de la conexión seleccionada:
	
![Selección de archivo parametros](/blog/applications/baudbstudio/manual/040-scripts-sql/barra-de-herramientas-seleccion-archivo-parametros.jpg "Selección de archivo parametros en la barra de herramientas")
			
Este botón nos abre una ventana donde podemos seleccionar el archivo de parámetros que acabamos de escribir (por ahora,
podemos dejar vacío el archivo de contexto ETL que se utiliza con los [scripts de ETL](/blog/applications/baudbstudio/manual/120-scripts-etl/120-scripts-etl):

![Selección de archivo de parámetros](/blog/applications/baudbstudio/manual/040-scripts-sql/seleccion-archivo-parametros.jpg "Selección de archivo de parámetros")

Una vez seleccionado el archivo de parámetros, nos aparecerá el nombre de archivo en la barra de herramientas:

![Visualizar el archivo de parámetros](/blog/applications/baudbstudio/manual/040-scripts-sql/barra-de-herramientas-archivo-parametros.jpg "Barra de herramientas: visualizar el archivo de parámetros")

Al pulsar el nombre del archivo de parámetros, se nos abrirá una ventana de edición del archivo en la que podemos hacer las
modificaciones necesarias:

![Modificación del archivo de parámetros](/blog/applications/baudbstudio/manual/040-scripts-sql/ver-archivo-parametros.jpg "Modificación del archivo de parámetros")
	
Ahora ya podemos ejecutar la consulta con el argumento seleccionado, por supuesto, podemos modificar estos parámetros cuando
queramos y tener tantos archivos de parámetros como necesitemos.

## Constantes

Al comienzo de este artículo, hemos utilizado una consulta como ésta para crear una tabla a partir de un archivo
parquet:
	
```SQL
DROP TABLE IF EXISTS Sales
GO

CREATE TABLE Sales AS
	SELECT *
		FROM parquet.`/mnt/c/Test/BauDbStudio/Samples/Files/Input/Sales.parquet`
GO
```

No va a poder ejecutar esta consulta en su ordenador a menos que tenga exactamente el mismo directorio. Esto
nos dará problemas cuando trabajamos en un equipo puesto que obligamos a todos sus componentes a tener los archivos
en los mismos directorios. Además, cuando [distribuyamos](/blog/applications/baudbstudio/manual/025-distribuciones/025-distribuciones)
nuestros scripts a otros entornos como DataBricks, debemos asegurarnos de cambiar estos directorios.
	
Para evitarlo, utilizaremos un parámetro especial, una constante en realidad.

Para definir una constante, utilizaremos el mismo archivo de parámetros que teníamos hasta ahora, pero le vamos a añadir
un nombre de parámetro especial, que debe comenzar por **Constant**, por ejemplo, voy a definir una constante
llamada **MountPath** que identifique el directorio base de mis archivos:
	
```JSON
{
	"Constant.MountPath": "/mnt/c/Test/BauDbStudio/Samples/Files",
	"StartDate": "2017-01-01"
}
```	

Por supuesto, puede utilizar el directorio que más le guste.

Ahora sólo tenemos que utilizar esta constante en nuestro script utilizando un nombre de variable especial rodeada
por dobles llaves: **{{MountPath}}**:
	
```SQL
DROP TABLE IF EXISTS Sales
GO

CREATE TABLE Sales AS
	SELECT *
		FROM parquet.`{{MountPath}}/Input/Sales.parquet`
GO
```

A partir de ahora, cada miembro del equipo puede almacenar los archivos donde quiera simplemente cambiando el archivo de parámetros. 

**Nota:** en mi caso he dejado el directorio de MountPath una carpeta por encima del directorio *Input*, es
una costumbre, me permite utilizar diferentes subdirectorios para mis archivos. Por ejemplo, puedo tener un directorio *Input*
para mis archivos de entrada y otro archivo *Output* con las tablas de salida y sólo tengo que cambiar los subdirectorios de
las consultas, no hace falta añadir nuevas constantes.

## Creación de una base de datos

En nuestro ejemplo de script con Spark SQL, estamos utilizando la base de datos **default**, no hay ningún problema con ello
pero, al mí al menos, me resulta más cómodo tener diferentes bases de datos para mis diferentes proyectos, así que nos
vamos a crear un script de creación de una nueva base de datos:
	
```SQL
-- Elimina la base de datos si existía
DROP DATABASE IF EXISTS DbSample CASCADE
GO

-- Crea la base de datos de cálculo
CREATE DATABASE IF NOT EXISTS DbSample
	LOCATION '{{MountPath}}/DbSample'
GO
```

En este caso la he llamado **DbSample**.

Pero imaginemos que estoy trabajando con diferentes scripts, me interesa que ese nombre de base de datos sea diferente para
todos los proyectos pero no quiero obligar a mis compañeros a utilizar un nombre único. Puedo utilizar otra constante,
por ejemplo, **DbCompute**. Ya nos podemos imaginar que eso se hace modificando el archivo de parámetros:
	
```JSON
{
	"Constant.DbCompute": "DbSample",
	"Constant.MountPath": "/mnt/c/Test/BauDbStudio/Samples/Files",
	"StartDate": "2017-01-01"
}
```	

Ahora puedo modificar el script de creación de base de datos para que utilice la constante:

```SQL
-- Elimina la base de datos si existía
DROP DATABASE IF EXISTS {{DbCompute}} CASCADE
GO

-- Crea la base de datos de cálculo
CREATE DATABASE IF NOT EXISTS {{DbCompute}} 
	LOCATION '{{MountPath}}/{{DbCompute}} '
GO
```

Cuando lo ejecute me va a crear una base de datos llamada **DbSample** en mi servidor y un directorio también llamado **DbSample**
por debajo de mi directorio **MountPath** donde Spark deja sus archivos.
	
Por supuesto, debo cambiar también el script de creación de tablas para que se ejecuten sobre esa base de datos:

```SQL
DROP TABLE IF EXISTS {{DbCompute}}.Sales
GO

CREATE TABLE {{DbCompute}}.Sales AS
	SELECT *
		FROM parquet.`/mnt/c/Test/BauDbStudio/Samples/Files/Input/Sales.parquet`
GO

DROP TABLE IF EXISTS {{DbCompute}}.SalesSummary
GO

CREATE TABLE {{DbCompute}}.SalesSummary AS
	SELECT StoreId, ProductId, SUM(Units) AS Units
		FROM Sales
		WHERE Date >= $StartDate
		GROUP BY StoreId, ProductId
GO
```

Una vez ejecutado, aparecerá la nueva base de datos bajo mi conexión:

![Base de datos DBSample](/blog/applications/baudbstudio/manual/040-scripts-sql/conexion-dbsample.jpg "Base de datos DBSample")

Las constantes que hemos visto, son muy importantes no sólo en la ejecución de scripts si no también en la 
[distribución de nuestros scripts](/blog/applications/baudbstudio/manual/025-distribuciones/025-distribuciones) a DataBricks. 