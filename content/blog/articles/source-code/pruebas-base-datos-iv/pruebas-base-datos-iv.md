+++
title = "Pruebas base datos - Exportación e importación"
date = "2019-10-11"
description = "Exportación e importación de base de datos a CSV"
thumbnail = "/articles/source-code/pruebas-base-datos/pruebas-base-datos.jpg"
tags = [ "Programación" ]
+++

Una de las funciones de la librería de base de datos es exportar e importar archivos CSV.

En esta ocasión, veremos las instrucciones para importar y exportar archivos.

## Modificaciones en el archivo de pasos

El archivo de definición de pasos cambia ligeramente con respecto a los que habíamos visto hasta ahora:

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Name>Test export data</Name>
	<PathProject>C:\Test\TestSamples\Scripts\CsvExport\</PathProject>
	<PathOutputFiles>C:\Test\TestSamples\Scripts\CsvExport\Output\</PathOutputFiles>
	<PathInputFiles>C:\Test\TestSamples\Scripts\CsvExport\Input\</PathInputFiles>

	<!-- Exporta los datos de ventas a archivos CSV -->
	<Step StartWithPreviousError = "false">
		<Name>Export to CSV</Name>
		<Script>Steps\05_Export_Csv.xml</Script>
	</Step>
</DbScript>
```

Aparecen dos nodos nuevos `PathOutputFiles` y `PathInputFiles` que identifican los directorios donde se van
a leer o escribir los archivos.

## Exportación de archivos CSV

Para exportar archivos, utilizamos la instrucción `ExportCsv`. En esta instrucción, indicamos los parámetros
del archivo de salida así como las columnas del archivo.

Por ejemplo, el siguiente script crea cuatro archivos, uno para cada una de las tablas que creamos en
los ejemplos del [artículo anterior](/blog/articles/source-code/pruebas-base-datos-iii/pruebas-base-datos-iii):

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Block Name="Export data to CSV">
		<ExportCsv Source="Sales" FileName="Stores.csv"
				   FieldsSeparator="," DecimalSeparator="." DateFormat="yyyy-MM-dd" ValueTrue="1" ValueFalse="0">
			<Column Name="StoreId" Type="Numeric" />
			<Column Name="Name" Type="String" />
			<Load>
				SELECT StoreId, Name
					FROM Stores
			</Load>
		</ExportCsv>

		<ExportCsv Source="Sales" FileName="Products.csv"
				   FieldsSeparator="," DecimalSeparator="." DateFormat="yyyy-MM-dd" ValueTrue="1" ValueFalse="0">
			<Column Name="ProductId" Type="Numeric" />
			<Column Name="Name" Type="String" />
			<Column Name="Price" Type="Numeric" />
			<Load>
				SELECT ProductId, Name, Price
				  FROM Products
			</Load>
		</ExportCsv>

		<ExportCsv Source="Sales" FileName="Sales.csv"
				   FieldsSeparator="," DecimalSeparator="." DateFormat="yyyy-MM-dd" ValueTrue="1" ValueFalse="0">
			<Column Name="SaleId" Type="Numeric" />
			<Column Name="StoreId" Type="Numeric" />
			<Column Name="ProductId" Type="Numeric" />
			<Column Name="Date" Type="DateTime" />
			<Column Name="Units" Type="Numeric" />
			<Column Name="Price" Type="Numeric" />
			<Load>
				SELECT SaleId, StoreId, ProductId, Date, Units, Price
					FROM Sales
			</Load>
		</ExportCsv>

		<ExportCsv Source="Sales" FileName="SalesGroupedByStore.csv"
				   FieldsSeparator="," DecimalSeparator="." DateFormat="yyyy-MM-dd" ValueTrue="1" ValueFalse="0">
			<Column Name="StoreId" Type="Numeric" />
			<Column Name="Units" Type="Numeric" />
			<Column Name="Price" Type="Numeric" />
			<Load>
				SELECT StoreId, Units, Price
					FROM SalesGroupedByStore
			</Load>
		</ExportCsv>
	</Block>
</DbScript>
```

En los atributos de la instrucción definimos:

* La conexión origen de los datos (atributo `Target`).
* El nombre del archivo de salida (atributo `FileName`). Recordemos que el nombre de archivo se compone con el nombre
definido en este atributo junto con el valor del nodo `PathOutputFiles` del archivo de pasos. 
* El carácter de separación entre campos (atributo `FieldsSeparator`).
* El carácter del punto decimal (atributo `DecimalSeparator`).
* El formato de las fechas (atributo `DateFormat`).
* Las cadenas utilizadas en los diferentes valores boolean (atributos `ValueTrue` y `ValueFalse`).

Dentro de la instrucción, definimos los nombres de las columnas que se van a utilizar en la cabecera del archivo y su tipo con
las etiquetas `Column`.

El último nodo (`Load`) es el contenido de la sentencia de consulta que actúa como origen de datos.

Como ejemplo, la instrucción de generación del archivo **Sales.csv** nos crea algo así:

```TXT
SaleId,StoreId,ProductId,Date,Units,Price
1,1,2,2019/02/18,10,70.2600676158969
2,1,5,2019/02/18,10,85.2951461321474
3,1,8,2019/02/18,10,34.7928229423792
4,1,9,2019/02/18,10,41.4442250625432
6,1,11,2019/02/18,3,25.6859830038019
```

Un pequeño apunte sobre el rendimiento: para un millón y medio de registros, el archivo se crea un unos quince segundos.

**Nota:** la definición de columnas, en la exportación realmente no debería ser necesaria y posiblemente
se elimine en las versiones finales.

## Importación de archivos CSV

La sentencia de importación de archivos `ImportCsv` es muy similar a la sentencia anterior:

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Block Name="Export data to CSV">
		<ImportCsv Target="Test" FileName="Stores.csv" Table="Stores"
				   SkipFirstLine="true" FieldsSeparator="," DecimalSeparator="." DateFormat="yyyy-MM-dd" ValueTrue="1">
			<Column Name="StoreId" Type="Numeric" />
			<Column Name="Name" Type="String" />
		</ImportCsv>

		<ImportCsv Target="Test" FileName="Products.csv" Table="Products"
				   SkipFirstLine="true" FieldsSeparator="," DecimalSeparator="." DateFormat="yyyy-MM-dd" ValueTrue="1">
			<Column Name="ProductId" Type="Numeric" />
			<Column Name="Name" Type="String" />
			<Column Name="Price" Type="Numeric" />
		</ImportCsv>

		<ImportCsv Target="Test" FileName="Sales.csv" Table="Sales"
				   SkipFirstLine="true" FieldsSeparator="," DecimalSeparator="." DateFormat="yyyy-MM-dd" ValueTrue="1">
			<Column Name="SaleId" Type="Numeric" />
			<Column Name="StoreId" Type="Numeric" />
			<Column Name="ProductId" Type="Numeric" />
			<Column Name="Date" Type="DateTime" />
			<Column Name="Units" Type="Numeric" />
			<Column Name="Price" Type="Numeric" />
		</ImportCsv>

		<ImportCsv Target="Test" FileName="SalesGroupedByStore.csv" Table="TestSalesGroupedByStore"
				   SkipFirstLine="true" FieldsSeparator="," DecimalSeparator="." DateFormat="yyyy-MM-dd" ValueTrue="1" ValueFalse="0">
			<Column Name="StoreId" Type="Numeric" />
			<Column Name="Units" Type="Numeric" />
			<Column Name="Price" Type="Numeric" />
		</ImportCsv>
	</Block>
</DbScript>
```

En este caso, en los atributos definimos: 

* La conexión destino de los datos (atributo `Target`).
* El nombre del archivo de entrada (atributo `FileName`). Recordemos que el nombre de archivo se compone con el nombre
definido en este atributo junto con el valor del nodo `PathInputFiles` del archivo de pasos.
* La tabla donde insertaremos los datos (atributo `Table`). En las importaciones no definimos ninguna consulta de lectura
si no una tabla física sobre la que se ejecutará un BulkCopy con los datos del archivo.
* Si la primera línea del archivo CSV es una cabecera (atributo `SkipFirstLine`).
* El carácter de separación entre campos (atributo `FieldsSeparator`).
* El carácter del punto decimal (atributo `DecimalSeparator`).
* El formato de las fechas (atributo `DateFormat`).
* Las cadenas utilizadas en los diferentes valores boolean (atributos `ValueTrue` y `ValueFalse`).

Además indicamos en los nodos `Column` las columnas y los tipos que van a recoger los datos en la tabla.

Para aquellos preocupados por el rendimiento: un millón de registros se insertan en la tabla en unos diez segundos.

## Conclusión

Este artículo lo hemos dedicado exclusivamente a las importaciones y exportaciones, siento no cumplir con las expectativas
generadas en el artículo anterior y no tratar otro tipo de sentencias pero esta parte parecía algo más lógica para los entornos
de pruebas donde en muchas ocasiones en lugar de bases de datos tendremos archivos de cálculos anteriores.