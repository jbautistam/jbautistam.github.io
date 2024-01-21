+++
title = "Pruebas base datos - datos de prueba"
date = "2019-10-11"
description = "Preparación de datos de prueba aleatorios para nuestras bases de datos"
thumbnail = "/articles/source-code/pruebas-base-datos/pruebas-base-datos.jpg"
tags = [ "Programación" ]
+++

En el [artículo anterior](/blog/articles/source-code/pruebas-base-datos-ii/pruebas-base-datos-ii), expliqué cómo utilizar
scripts para la creación de dos bases de datos: una es la semilla para nuestras pruebas y otra
sobre la que ejecutaremos las sentencias de test.

En este artículo veremos cómo crear los datos de nuestra base de datos semilla y copiarlos a la base de datos de prueba.

## Procesos de ejecución
	
El proceso de ejecución en este ejemplo es similar al del artículo anterior, de hecho vamos a utilizar las mismas conexiones, 
pero le vamos a añadir dos procesos nuevos:
	
```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Name>Test copy data</Name>
	<PathProject>C:\Test\Scripts\CopyData</PathProject>

	<!-- Creación de la base de datos SalesDb -->
	<Step StartWithPreviousError = "false">
		<Name>Scripts database process: create database SalesDb</Name>
		<Script>DbScripts\01_Create_Database_Sales.xml</Script>
		<Parameter Key="ConnectionName" Type="string" Value="Sales" />
		<Parameter Key="DbName" Type="string" Value="SalesDb" />
	</Step>

	<!-- Creación de la base de datos SalesDbTesting -->
	<Step StartWithPreviousError = "false">
		<Name>Scripts database process: create database SalesDbTesting</Name>
		<Script>DbScripts\01_Create_Database_Sales.xml</Script>
		<Parameter Key="ConnectionName" Type="string" Value="Test" />
		<Parameter Key="DbName" Type="string" Value="SalesDbTesting" />
	</Step>

	<!-- Inserción de datos de prueba sobre la base de datos SalesDb -->
	<Step StartWithPreviousError = "false">
		<Name>Scripts database process: create test data</Name>
		<Script>DbScripts\05_Sales_Insert_Data.xml</Script>
		<Parameter Key="StartDate" Type="dateTime" Value="Now" Increment="-4" Interval="month" Mode="PreviousMonday" />
		<Parameter Key="EndDate" Type="dateTime" Value="Now" Increment="1" Interval="day" Mode="NextMonday" />
	</Step>

	<!-- Copia los datos de SalesDb a SalesDbTest -->
	<Step StartWithPreviousError = "false">
		<Name>Scripts database process: copy sales data</Name>
		<Script>DbScripts\10_Copy_Sales_Into_Test.xml</Script>
		<Parameter Key="StartDate" Type="dateTime" Value="Now" Increment="-2" Interval="month" Mode="PreviousMonday" />
		<Parameter Key="EndDate" Type="dateTime" Value="Now" Increment="1" Interval="day" Mode="NextMonday" />
	</Step>
</DbScript>
```

Los últimos dos procesos ejecutan los scripts `05_Sales_Insert_Data.xml` que crea datos para la base de datos semilla y 
`10_Copy_Sales_Into_Test.xml` que copia los datos a la base de datos de prueba. Detallaremos el
contenido de estos archivos en breve.

La única modificación importante con respecto a los procesos anteriores es que hemos añadido dos parámetros,
`StartDate` y `EndDate`. 

Ambos parámetros son de tipo fecha y el valor es la fecha actual con un incremento variable. Por ejemplo,
la primera definición de la variable `StartDate`:
	
```XML
<Parameter Key="StartDate" Type="dateTime" Value="Now" Increment="-4" Interval="month" Mode="PreviousMonday" />
```

toma como valor la fecha de hoy, le resta cuatro meses (atributos `Increment` e `Interval`) y lo pasa
al lunes anterior (atributo `Mode`).
	
Por su parte, el valor `EndDate` del primer proceso, recoge el primer lunes después de mañana:

```XML
<Parameter Key="EndDate" Type="dateTime" Value="Now" Increment="1" Interval="day" Mode="NextMonday" />
```

Si nos fijamos, los valores de `StartDate` son diferentes en los dos pasos. Es así porque en el segundo
paso no vamos a copiar todos los datos si no únicamente los que estén en el intervalo entre `StartDate` y
`EndDate` de este proceso (que no tienen porqué ser y de hecho no son, los mismos que en el paso anterior). 
	
Recuperaremos las definiciones de argumentos locales y globales en futuros artículos. 

## Creación de los datos de pruebas

Como decíamos en el [primer artículo de esta serie](/blog/articles/source-code/pruebas-base-datos/pruebas-base-datos), lo normal
al hacer pruebas de base de datos es utilizar una base de datos semilla pero para complicar un poco el ejemplo,
utilizaremos un script para generar los datos de forma más o menos aleatora y aprovechamos para introducir
las instrucciones `print` y `for`.
	
El archivo que genera lo datos, `05_Sales_Insert_Data.xml`, contiene el siguiente código:

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Block Name="Delete old data">
		<Execute Target="Sales">
			TRUNCATE TABLE Sales;
			DELETE FROM Products;
			DELETE FROM Stores;
		</Execute>
	</Block>

	<Block Name="Insert stores">
		<For Variable ="Index" Start="1" End="50">
			<Print>Inserting store {{Index}}</Print>
			<Execute Target="Sales">
				INSERT INTO Stores (Name)
					VALUES ('Store {{Index}}')
			</Execute>
		</For>
	</Block>

	<Block Name="Insert products">
		<For Variable ="Index" Start="1" End="300">
			<Print>Inserting product {{Index}}</Print>
			<Execute Target="Sales">
				INSERT INTO Products (Name, Price)
					VALUES ('Product {{Index}}', 100 * RAND())
			</Execute>
		</For>
	</Block>

	<Block Name="Insert sales">
		<For Variable ="Index" Start="StartDate" End="EndDate">
			<Print>Inserting sale {{Index}}</Print>
			<Execute Target="Sales">
				INSERT INTO Sales (StoreId, ProductId, Date, Units, Price)
					SELECT StoreId, ProductId, {{Index}}, Units, Price
						FROM (SELECT tmpKeys.StoreId, tmpKeys.ProductId, 
									 CONVERT(int, 10 + -10 * Random) AS Units,
									 tmpKeys.Price + 10 * Random AS Price
								FROM (SELECT Stores.StoreId, Products.ProductId, Products.Price, 
											 ABS(CAST(CAST(NEWID() AS VARBINARY) AS INT)) / ABS(CAST(CAST(NEWID() AS VARBINARY) AS INT)) * Rand() AS Random
										FROM Products CROSS JOIN Stores) AS tmpKeys) AS tmpRandom
						WHERE Units > 0
			</Execute>
		</For>
	</Block>
</DbScript>
```

El código es muy similar a los que hemos visto hasta ahora: cuatro bloques de código, el primero de ellos borra los
datos anteriores de las tablas, el segundo inserta datos sobre la tabla `Stores`, el tercero inserta datos
sobre `Products` y el último inserta datos sobre la tabla `Sales`.
	
**Nota:** por supuesto, dado que hemos definido integridad referencial a la hora de crear las tablas, el
orden de ejecución es importante.
	
El segundo bloque, introduce dos sentencias nuevas, `Print` y `For`:

```XML
<For Variable ="Index" Start="1" End="50">
	<Print>Inserting store {{Index}}</Print>
	<Execute Target="Sales">
		INSERT INTO Stores (Name)
			VALUES ('Store {{Index}}')
	</Execute>
</For>
```

La sentencia `for` es prácticamente igual que en el resto de lenguajes, simplemente definimos el nombre
de la variable con el valor del atributo `Variable` y los valores de inicio y fin con los atributos `Start` y
`End`. También se puede utilizar un atributo `Step` para el incremento del índice, si no se utiliza,
como en este caso, el valor del incremento es 1.

**Nota:** sí, podríamos haber utilizado T-Sql para ejecutar un bucle `WHILE` en 
lugar de crearnos una sentencia for externa, pero recordemos que la intención de este proyecto es ofrecer un marco
de pruebas para cualquier base de datos, no sólo para SQL Server.

La sentencia `Print` simplemente escribe un mensaje en la consola. Existe una diferencia sutil entre los mensajes que escriben 
las sentencias `Block` y `Print`, la primera de ellas lanza un evento de log de tipo infomativo mientras que `Print`
lanza un evento para imprimir en la consola (que no tiene porqué ser la misma que trata el log, dependerá del ejecutable que 
maneje la librería).

Por supuesto, en nuestros comandos podemos utilizar el contenido de la variable con la notación de dobles llaves que veíamos
en el artículo anterior:

```SQL
INSERT INTO Stores (Name)
	VALUES ('Store {{Index}}')
```

Con esta sentencia, insertamos 50 registros sobre la tabla `Stores` cuyo nombre será `Store {{Index}}`
donde `Index` se sustituirá por los valores del índice en el bucle (Store 1, Store 2...).
	
La inserción en la tabla `Products` es similar (aunque en este caso se insertan 300 registros), pero la inserción
sobre la tabla `Sales` es ligeramente diferente:
	
```XML
<For Variable ="Index" Start="StartDate" End="EndDate">
	<Print>Inserting sale {{Index}}</Print>
	<Execute Target="Sales">
		INSERT INTO Sales (StoreId, ProductId, Date, Units, Price)
			SELECT StoreId, ProductId, {{Index}}, Units, Price
				FROM (SELECT tmpKeys.StoreId, tmpKeys.ProductId, 
							 CONVERT(int, 10 + -10 * Random) AS Units,
							 tmpKeys.Price + 10 * Random AS Price
						FROM (SELECT Stores.StoreId, Products.ProductId, Products.Price, 
									 ABS(CAST(CAST(NEWID() AS VARBINARY) AS INT)) / ABS(CAST(CAST(NEWID() AS VARBINARY) AS INT)) * Rand() AS Random
								FROM Products CROSS JOIN Stores) AS tmpKeys) AS tmpRandom
				WHERE Units > 0
	</Execute>
</For>
```	

El bucle `for` de este proceso, en lugar de utilizar un índice con un número entero utiliza fechas. Concretamente
va desde el valor del parámetro `StartDate` hasta el valor `EndDate`. Por supuesto, estos son los mismos
valores que hemos definido como parámetros a la hora de ejecutar el paso.

La sentencia de inserción de datos es un poco más complicada que en el caso anterior pero no es más que SQL básico de
inserción de datos para todas las tiendas y todos los productos aleatoriamente para cada día del bucle. Para que no todas
las fechas tengan los mismos productos / tiendas, sólo se grabarán aquellos que tienen unidades (las unidades
son un valor aleatorio entre -10 y +10, se insertan sólo las que son mayor que 0).

En mis pruebas, este proceso genera unos 700.000 registros de ventas en unos veinte segundos.

**Nota:** todas las pruebas las he hecho con un ordenador con un procesador i7, 32GB de RAM
con SQL Server 2017 for Linux instalado sobre Docker.

## Copia de datos

El último script, `10_Copy_Sales_Into_Test.xml` simplemente copia los datos entre la base de datos `SalesDb`
donde hemos creado los datos y la base de datos `SalesDbTest` sobre las que vamos a ejecutar las pruebas:
	
```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Block Name="Delete old data">
		<Execute Target="Test">
			TRUNCATE TABLE Sales;
			DELETE FROM Products;
			DELETE FROM Stores;
		</Execute>
	</Block>

	<Block Name="Copying stores">
		<BulkCopy Source="Sales" Target="Test" Table="Stores">
			SELECT StoreId, Name
			  FROM Stores
		</BulkCopy>
	</Block>

	<Block Name="Copying products">
		<BulkCopy Source="Sales" Target="Test" Table="Products">
			SELECT ProductId, Name, Price
				FROM Products
		</BulkCopy>
	</Block>

	<Block Name="Copying sales">
		<BulkCopy Source="Sales" Target="Test" Table="Sales">
			SELECT SaleId, StoreId, ProductId, Date, Units, Price
			  FROM Sales
			  WHERE Date BETWEEN @StartDate AND @EndDate
		</BulkCopy>
	</Block>
</DbScript>
```

Muy simple, una serie de instrucciones para borrar los datos antiguos y para copiar datos de tablas como ya vimos en el primer
artículo de la serie.

El único comando ligeramente diferente es la última instrucción `BulkCopy`:

```XML
<BulkCopy Source="Sales" Target="Test" Table="Sales">
	SELECT SaleId, StoreId, ProductId, Date, Units, Price
	  FROM Sales
	  WHERE Date BETWEEN @StartDate AND @EndDate
</BulkCopy>
```

En este caso, los datos de la tabla `Sales` se van a filtrar por las fechas de inicio y fin pasadas como parámetros en
el paso.

Observad que para indicar los argumentos en la sentencia SQL hemos utilizado la notación general de base de datos 
(`@StartDate` y `@EndDate`) en lugar de la notación de doble llave que veníamos utilizado hasta ahora.

En realidad, en el caso de los comandos de base de datos, se pueden utilizar ambas notaciones. La única diferencia es que 
con la notación de doble llave, el motor sustituye el valor de las variables y podríamos indicar diferentes formatos
mientras que con la notación de base de datos simplemente se añade como parámetro al comando que se lanza
sobre la base de datos sin sustituir el valor en la cadena SQL.

Aunque aún es demasiado pronto para hablar de rendimiento, para los curiosos, el tiempo de copia de 700.000 registros entre ambas bases de
datos está en torno a los 8 segundos. Por supuesto, la velocidad se debe a la instrucción bulkcopy de SQL Server no a mis optimizaciones y
estamos dentro del mismo servidor por lo que no hay tiempos añadidos de transmisión de datos.

### Conclusión

En este artículo hemos añadido la posibilidad de crear datos al vuelo utilizando sentencias `for` y hemos visto algún ejemplo
de uso de parámetros.

En los siguientes artículos profundizaremos en otros aspectos del motor como los cursores sobre datos, las sentencias `if` y `while` y posiblemente
las operaciones matemáticas y condicionales.
