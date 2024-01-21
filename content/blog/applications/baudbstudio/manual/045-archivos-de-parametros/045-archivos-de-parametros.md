+++
title = "Archivos de parámetros"
date = "2019-10-11"
description = "Archivos de parámetros de ejecución de scripts y consultas de BauDbStudio"
thumbnail = "/applications/baudbstudio/manual/045-archivos-de-parametros/045-archivos-de-parametros.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

Cuando escribimos scripts o consultas SQL, en ocasiones necesitamos asociarle parámetros.

Por ejemplo, esta consulta sobre una base de datos SqLite:

```SQL
SELECT [Orders].[OrderID], [Orders].[CustomerID], [Orders].[EmployeeID], [Orders].[OrderDate], 
		[Orders].[RequiredDate], [Orders].[ShippedDate], [Orders].[ShipVia], 
		[Orders].[Freight], [Orders].[ShipName], [Orders].[ShipAddress], [Orders].[ShipCity], 
		[Orders].[ShipRegion], [Orders].[ShipPostalCode], [Orders].[ShipCountry]
	FROM [Orders]
	WHERE OrderDate > @StartDate
```

muestra aquellos pedidos posteriores a determinada fecha, llamada **StartDate**.

Para que esta consulta funcione, se le debe pasar el valor del parámetro **StartDate** a la consulta SQL, si no, el servidor
de base de datos dará un error.
	
Para no tener que introducir estos valores manualmente, **BauDbStudio** utiliza los archivos de parámetros.

Un archivo de parámetros es un archivo JSON que contiene todos los parámetros que vamos a utilizar en nuestro script o
consulta, por ejemplo, en nuestro caso nuestro archivo sería algo así:
	
```JSON
{
	"StartDate": "1970-01-01"
}
```

Por supuesto, nuestro archivo de parámetro puede contener todos los parámetros que deseemos.

Podemos tener varios archivos de parámetros en nuestro ordenador, por ejemplo asociados a diferentes clientes o proyectos, pero
sólo podemos tener un archivo de parámetros activo. Este archivo de parámetro lo seleccionamos pulsando el botón 
**Seleccionar archivo de parámetros** de la barra de herramientas principal.
	
Una vez seleccionado un archivo de parámetros, veremos su nombre en la barra de herramientas junto a la conexión y al pulsar
sobre él podemos editar su contenido:
	
![Selección de archivo de parámetros](/blog/applications/baudbstudio/manual/045-archivos-de-parametros/seleccionarchivoparametros.jpg "Selección de archivo de parámetros")
		   
Dado que es bastante común cambiar de archivo de parámetros, **BauDbStudio** mantiene una lista con los últimos archivos de parámetros
seleccionados para que podamos cambiar de uno a otro rápidamente.
	
Aparte de los argumentos de consulta, en los archivos de parámetros podemos definir constantes como por ejemplo
un nombre de base de datos o un directorio de archivo de datos. Por ejemplo, en este código:
	
```JSON
{
	"Constant.DbCompute": "DbSample",
	"Constant.MountPath": "/mnt/c/Test/BauDbStudio/Samples/Files",
	"StartDate": "1970-01-01"
}
```

Definimos dos contantes, la primera de ellas **DbCompute** es un nombre de base de datos mientras que **MountPath** es el nombre
de un directorio.
	
Podemos utilizar estas constantes por ejemplo cuando insertamos los datos de un archivo parquet en una tabla utilizando Spark Sql:

```SQL
CREATE TABLE {{DbCompute}}.Data
	SELECT `Id`, TRIM(`Name`) AS `Name`
		FROM parquet.`{{MountPath}}/2020-07-26/DataFile.parquet`
```

Esta consulta crea una tabla llamada **Data** sobre la base de datos **DbSample** (el valor definido en la constante **DbCompute**)
con el contenido del archivo **DataFile.parquet** que está en el directorio
*/mnt/c/Test/BauDbStudio/Samples/Files/2020-07-26*, es decir, el contenido de la constante **MountPath** combinado
con la cadena *2020-07-26*.

**Nota:** puede encontrar más información sobre parámetros en la página dedicada a los
[scripts de SQL](/blog/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql).
