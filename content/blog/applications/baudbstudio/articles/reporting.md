+++
title = "BauDbStudio: SQL para generación de informes"
date = "2020-12-06"
description = "Cómo utilizar BauDbStudio como herramienta de visualización de informes"
thumbnail = "/applications/baudbstudio/images/BauDbStudio.png"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

Una de las cosas que más envidio de las herramientas de Bussiness Intelligence o reporting es su capacidad para realizar consultas relativamente
complejas prácticamente sin programación.

Por eso, estos últimos meses he estado trabajando en algunas ideas para reproducir este comportamiento en **BauDbStudio**.

Normalmente la bases de datos utilizadas con reporting son especiales. Los datos se almacenan de forma que la explotación sea lo más sencilla posible,
bien denormalizando tablas o bien agrupando datos en tablas para que el acceso sea más rápido.

Sobre estas tablas se genera además una estructura de metadatos con nombres y relaciones entre tablas para que el usuario pueda seleccionar los informes
y consultas directamente desde la interface de usuario.

## Generación de base de datos de reporting

Lo primero que tenemos que hacer, por tanto, es generar una base de datos de reporting. Esto no es completamente necesario, podríamos crear
los informes y metadatos sobre una base de datos *normal*, pero lo habitual es dejarlo en bases de datos separadas.

Como ejemplo, he cogido la base de datos **Northwind**, la he copiado sobre una base de datos SqLite simplemente por demostrar que en algunos casos
no es necesario utilizar un servidor de base de datos avanzado como SQL Server o Spark y he generado una base de datos de reporting también con SqLite con tablas preprocesadas
para que podamos generar informes.

En este punto, ya nos aparece un concepto básico de los procesos de BI, el proceso de ETL (Extract, Transform and Load). Un ETL es un proceso automático que copia 
y transforma datos entre bases de datos y/o archivos.

Para este paso, he utilizado también **BauDbStudio**. Para ello me he generado algunos archivos de script que realizan la copia y agregación de datos de la
base de datos inicial a la base de datos de reporting. 

Este caso es muy sencillo, pero sólo para que nos sirva de referencia, en el primer paso creamos las tablas en la base de datos destino:

```XML
<?xml version="1.0" encoding="utf-8" ?>
<DbScript>
	<Block Message="Create dimensions tables">
		<Execute Target="Reporting">
			DROP TABLE IF EXISTS [DimLocations];
			CREATE TABLE IF NOT EXISTS [DimLocations]
			(
				[LocationId] INTEGER NOT NULL,
				[Country] nvarchar(200) NOT NULL,
				[City] nvarchar(200) NULL,
				[Region] nvarchar(200) NULL,
				[PostalCode] nvarchar(200) NULL,
				CONSTRAINT [PK_Locations] PRIMARY KEY ([LocationId])
			);

			DROP TABLE IF EXISTS [DimCustomers];
			CREATE TABLE IF NOT EXISTS [DimCustomers]
			(
				[CustomerId] INTEGER NOT NULL,
				[Code] nvarchar(20) NOT NULL,
				[Name] nvarchar(200) NOT NULL,
				[LocationId] INTEGER,
				CONSTRAINT [PK_Customer] PRIMARY KEY ([CustomerId])
			);
		</Execute>
	</Block>
			
	<Block Message="Create facts tables">
		<Execute Target="Reporting">
			DROP TABLE IF EXISTS [FactSales];
			CREATE TABLE IF NOT EXISTS [FactSales]
			(
				[CustomerId] INTEGER NOT NULL,
				[EmployeeId] INTEGER NOT NULL,
				[ProductId] INTEGER NOT NULL,
				[LocationId] INTEGER NOT NULL,
				[Date] Date NOT NULL,
				[Price] float NOT NULL
			);
		</Execute>
	</Block>
</DbScript>
```

El segundo paso, copia datos y en su caso los agrupa:

```XML
<?xml version="1.0" encoding="utf-8" ?>
<DbScript>
	<Block Message="Copying locations">
		<Execute Target="Reporting">
			DELETE FROM DimLocations
		</Execute>

		<BulkCopy Source="Northwind" Target="Reporting" Table="DimLocations">
			SELECT ROW_Number() OVER (ORDER BY Country) AS LocationId, Country, City, Region, PostalCode
				FROM (SELECT Country, City, Region, PostalCode
					  	FROM Suppliers
					  UNION
					  SELECT Country, City, Region, PostalCode
					  	FROM Employees
					  UNION
					  SELECT Country, City, Region, PostalCode
					  	FROM Customers
					  UNION
					  SELECT ShipCountry, ShipCity, ShipRegion, ShipPostalCode
					  	FROM Orders
					)
		</BulkCopy>
	</Block>
	
	<Block Message="Copying customers">
		<Execute Target="Reporting">
			DROP TABLE IF EXISTS [SyncDimCustomers];
			CREATE TABLE IF NOT EXISTS [SyncDimCustomers]
			(
				[CustomerId] nvarchar(5) NOT NULL,
				[Name] nvarchar(200) NOT NULL,
				[Country] nvarchar(200) NOT NULL,
				[City] nvarchar(200) NULL,
				[Region] nvarchar(200) NULL,
				[PostalCode] nvarchar(200) NULL,
				CONSTRAINT [PK_SynCustomers] PRIMARY KEY ([CustomerId])
			);
		</Execute>

		<BulkCopy Source="Northwind" Target="Reporting" Table="SyncDimCustomers">
			SELECT CustomerID, CompanyName AS Name, Country, City, Region, PostalCode
				FROM Customers
		</BulkCopy>

		<BulkCopy Source="Reporting" Target="Reporting" Table="DimCustomers">
			SELECT ROW_Number() OVER (ORDER BY SyncDimCustomers.CustomerId) AS CustomerId, SyncDimCustomers.CustomerId AS Code, 
				   SyncDimCustomers.Name, DimLocations.LocationId
				FROM SyncDimCustomers INNER JOIN DimLocations
					ON IfNull(SyncDimCustomers.Country, '') = IfNull(DimLocations.Country, '')
						AND IfNull(SyncDimCustomers.Region, '') = IfNull(DimLocations.Region, '')
						AND IfNull(SyncDimCustomers.City, '') = IfNull(DimLocations.City, '')
						AND IfNull(SyncDimCustomers.PostalCode, '') = IfNull(DimLocations.PostalCode, '')
		</BulkCopy>
		
		<Execute Target="Reporting">
			DROP TABLE IF EXISTS [SyncDimCustomers];
		</Execute>
	</Block>
	
	<Block Message="Copying fact sales">
		<Execute Target="Reporting">
			DROP TABLE IF EXISTS [SyncFactSales];
			CREATE TABLE IF NOT EXISTS [SyncFactSales]
			(
				[CustomerCode] nvarchar(5) NOT NULL,
				[EmployeeId] INTEGER NOT NULL,
				[ProductId] INTEGER NOT NULL,
				[Date] Date NOT NULL,
				[Country] nvarchar(200) NOT NULL,
				[City] nvarchar(200) NULL,
				[Region] nvarchar(200) NULL,
				[PostalCode] nvarchar(200) NULL,
				[Price] float NOT NULL
			);
		</Execute>

		<BulkCopy Source="Northwind" Target="Reporting" Table="SyncFactSales">
			SELECT Orders.CustomerId AS CustomerCode, Orders.EmployeeId, Details.ProductId, 
				   IfNull(Orders.ShippedDate, Orders.OrderDate) AS Date, Orders.ShipCountry AS Country,
				   Orders.ShipRegion AS Region, Orders.ShipCity AS City, Orders.ShipPostalCode AS PostalCode,
				   Details.UnitPrice * Details.Quantity - Details.Discount AS Price
			  FROM [Orders] INNER JOIN [Order Details] AS Details
			  	ON Orders.OrderId = Details.OrderId
		</BulkCopy>

		<BulkCopy Source="Reporting" Target="Reporting" Table="FactSales">
			SELECT DimCustomers.CustomerId, DimEmployees.EmployeeId, DimProducts.ProductId, DimLocations.LocationId,
					SyncFactSales.Date, SyncFactSales.Price
				FROM SyncFactSales INNER JOIN DimCustomers
					ON SyncFactSales.CustomerCode = DimCustomers.Code
				INNER JOIN DimEmployees
					ON SyncFactSales.EmployeeId = DimEmployees.EmployeeId
				INNER JOIN DimProducts
					ON SyncFactSales.ProductId = DimProducts.ProductId
				INNER JOIN DimLocations
					ON IfNull(SyncFactSales.Country, '') = IfNull(DimLocations.Country, '')
						AND IfNull(SyncFactSales.Region, '') = IfNull(DimLocations.Region, '')
						AND IfNull(SyncFactSales.City, '') = IfNull(DimLocations.City, '')
						AND IfNull(SyncFactSales.PostalCode, '') = IfNull(DimLocations.PostalCode, '')
		</BulkCopy>
		
		<Execute Target="Reporting">
			DROP TABLE IF EXISTS [SyncFactSales];
		</Execute>
	</Block>
</DbScript>
```

Este último paso parece más complejo pero simplemente crea tablas temporales, copia los datos de la base de datos inicial y agrupa los datos necesarios.

Fijáos que en la base de datos de reporting se están creando códigos al vuelo como por ejemplo *CustomerId*, las claves en nuestra base de datos de reporting
no tienen porqué ser las mismas que en nuestras bases de datos OLTP y muchas veces no lo serán. Por ejemplo, puedo borrar un cliente en mi base de datos OLTP
pero debo mantenerlo en mi base de datos de reporting porque sus ventas ya están agrupadas y contabilizadas y debemos mantenerlas a lo largo del tiempo.

En un ejemplo tan sencillo como éste se están recreando completamente las claves en cada importación, en casos más complejos habría que modificar los scripts
para que mantenga las claves. Como ejemplo, nos basta así.

Una vez importados los datos, nuestra base de datos de reporting tiene este esquema:

![Esquema de base de datos](/blog/applications/baudbstudio/articles/ReportingDbSchema.jpg "Esquema de la base de datos de informes")

Si nos fijamos, tenemos dos tipos de nombre en las tablas, unas que empiezan por *dim* y otras que comienzan por *fact*.

Las tablas que comienzan por *fact* se llaman tablas de hechos, en una [tabla de hechos](https://es.wikipedia.org/wiki/Tabla_de_hechos) se
almacenan los valores de negocio y las relaciones con las dimensiones que la definen.

Por su parte, Las tablas que comienzan por *dim* son tablas de dimensiones, en una [tabla de dimensión](https://es.wikipedia.org/wiki/Tabla_de_dimensi%C3%B3n) 
se almacenan atributos o campos utilizados para restringir las consultas de los datos almacenados en la tabla de hechos.

Por supuesto esta forma de nombrarlo no es obligatoria pero posteriormente nos servirán para identificarlas más fácilmente.

Lo habitual es que las tablas de hechos contengan únicamente valores o cantidades y campos de relación con las tablas de dimensión.

Por ejemplo, nuestra tabla de ventas contiene precios y fechas como valores y los códigos de cliente, vendedor, proveedor... que identifican las
diferentes dimensiones:

![Tabla FactSales](/blog/applications/baudbstudio/articles/FactSalesTable.jpg "Campos de la tabla FactSales")

Como podemos apreciar, los campos que terminan por *Id* son claves foráneas a las tablas de dimensión.

Una tabla de dimensión, como **DimCustomers** por ejemplo, almacena información y atributos sobre el cliente:

![Tabla DimCustomers](/blog/applications/baudbstudio/articles/DimCustomersTable.jpg "Campos de la tabla DimCustomers")

En este caso, tenemos el nombre del cliente y su código y una clave foránea a la tabla *DimLocations* llamada *LocationId*. Es decir,
una tabla de dimensión puede hacer referencia a otras tablas de dimensión, en nuestro caso la tabla **DimLocations** tiene estos
campos:

![Tabla DimLocations](/blog/applications/baudbstudio/articles/DimLocationsTable.jpg "Campos de la tabla DimLocations")

Eso nos permite crear un modelo en estrella o de copo de nieve entre nuestras tablas de dimensión y nuestras tablas de hechos.

![Modelo de copo de nieve](/blog/applications/baudbstudio/articles/SnowflakeModel.jpg "Modelo de copo de nieve")

## Definiendo metadatos

Hasta el momento, hemos creado nuestra base de datos de reporting y hemos introducido datos desde nuestra base de datos transaccional, pero
aún no hemos definido ningún metadato, por tanto, aún no podemos utilizarla para hacer consultas complejas.

Los metadatos nos van a permitir identificar:

* Las tablas que vamos a utilizar como dimensiones 
* La jerarquías entre las dimensiones
* Las tablas que vamos a utilizar como hechos y sus relaciones con las dimensiones

Estos metadatos, por tanto, son los que permitirán que el usuario pueda realizar consultas complejas.

Nuestra estructura de metadatos, queda de esta forma:

![Estructura de metadatos](/blog/applications/baudbstudio/articles/Metadata.jpg "Metadatos")

Para el ejemplo he creado una estructura sencilla:

* En **orígenes de datos** tenemos las tablas base del esquema sobre el que vamos a trabajar. En este caso
sólo tenemos tablas físicas pero se pueden definir también orígenes de datos utilizando SQL que se comportarán
como vistas en las relaciones.
* En **dimensiones** se definen las jerarquías que van a utilizar nuestros informes, por ejemplo vemos que la
dimensión de productos *Products dimension* lee del origen de datos *DimProducts* y se relaciona con las dimensiones
*Categories dimension* (que lee del origen de datos *DimCategories*) y la dimensión *DimSuppliers* que a la vez se
relaciona con la dimensión *Locations dimension*, es decir, un producto, aparte de sus datos básicos, tiene categorías
y proveedores.
* Por último, los **informes** como *Sales* leen de un origen de datos (en este caso *FactSales*, nuestra tabla de hechos) y tienen unas dimensiones
relacionadas, en este caso *Customers dimension*, *Employees dimension*, *Products dimension* y *Location dimension*. Es
decir, podremos consultar datos por cliente, por empleado, por producto y por ubicación de venta y dado que los productos
están relacionados con categorías también por la categoría del producto y dado que está relacionado con proveedores podríamos
hacer consultas por la ciudad del proveedor (dado que los proveedores se relacionan con ubicaciones ¿queda clara la idea?).

Por supuesto, estos metadatos se pueden modificar o ampliar utilizando **BauDbStudio** pero este artículo es para explicar
las posibilidades, no vamos a entrar en el mantenimiento de los metadatos.

## Ejecución de informes

Si en **BauDbStudio** ejecutamos una consulta sobre un informe definido:

![Consulta sobre el informe de ventas](/blog/applications/baudbstudio/articles/QuerySales.jpg "Consulta de informe")

Nos abrirá una pantalla de ejecución del informe:

![Ventana del informe de ventas](/blog/applications/baudbstudio/articles/QuerySalesWindow.jpg "Informe de ventas")

En el árbol de la izquierda podemos ver las dimensiones y expresiones que podemos utilizar en este informe.

En concreto, para el informe *Sales* hemos definido las dimensiones de clientes con sus ubicaciones, empleados con sus ubicaciones, ubicaciones (para el
lugar en el que se ha producido la venta) y los productos con sus categorías y proveedores con sus ubicaciones.

También tenemos dos expresiones en la parte inferior: la fecha y el precio de la venta.

Podemos hacer una consulta sencilla para empezar, simplemente veamos las fechas y precios de venta sin seleccionar niguna dimensión, para ello seleccionamos
las expresiones de fecha y precio y pulsamos sobre la ejecución de consulta. En la ventana nos aparecerá la SQL creada y el resultado de la consulta:

![Consulta sencilla de expresiones](/blog/applications/baudbstudio/articles/QuerySalesExpressions.jpg "Consulta de expresiones")

Vamos a hacer algo un poco más complicado, vamos a obtener la suma de precios agrupados por fecha y ordenados por fecha ascendente.
Para ello simplemente seleccionamos la agregación de suma y pulsamos sobre el botón de ejecución de consulta:

![Consulta de expresiones con suma](/blog/applications/baudbstudio/articles/QuerySalesExpressionsSum.jpg "Suma de precios ordenado por fecha")

Pero con expresiones es sencillo, hagamos algo más complejo, visualicemos el nombre de cliente, producto y la ciudad del cliente. Simplemente tenemos
que seleccionar los campos requeridos en el árbol y ejecutar:

![Consulta con dimensiones](/blog/applications/baudbstudio/articles/QuerySalesDimensions.jpg "Consulta con dimensiones")

Ahora filtremos por los clientes de *Lyon*, para ello añadimos un filtro sobre la ciudad del cliente:

![Filtro por la ciudad del cliente](/blog/applications/baudbstudio/articles/FilterCustomerCity.jpg "Filtro de clientes")

Y ejecutamos:

![Consulta con dimensiones filtrado](/blog/applications/baudbstudio/articles/QuerySalesDimensionsFiltered.jpg "Consulta filtrada con dimensiones")

Pero dado que nos van a salir todos los datos de *Lyon*, vamos a quitarle este campo del resultado y le vamos a añadir la categoría del producto:

![Consulta con categoría de producto](/blog/applications/baudbstudio/articles/QuerySalesDimensionsProductFiltered.jpg "Consulta filtrada con categoría")

Por último, nos puede interesar ver sólo las ventas con una suma total por fecha superior a 300. En ese caso añadimos un filtro sobre precios pero
en la parte del agregado, es decir, queremos una cláusula HAVING en nuestra consulta final. Para ésto añadimos el filtro al campo agregado:

![Filtro sobre la suma de precios](/blog/applications/baudbstudio/articles/FilterPriceHaving.jpg "Filtro HAVING")

Y ejecutamos:

![Consulta con filtro sobre la suma de precios](/blog/applications/baudbstudio/articles/QuerySalesDimensionsProductHaving.jpg "Consulta con un filtro para el total")

Por supuesto, también podemos hacer nuestro pequeño gráfico del resultado aunque aún no estoy muy orgulloso de él:

![Gráfico del resultado de la consulta](/blog/applications/baudbstudio/articles/QueryChart.jpg "Gráfico")

A partir de aquí, todo es jugar con ordenaciones, filtros, agrupaciones... para que nuestro usuario pueda obtener los datos sin necesidad de saber
SQL dado que nuestros informes están previamente preparados con una serie de metadatos para facilitarle el trabajo.

Por supuesto para nosotros, como programadores, la vida será más fácil porque no vamos a tener que escribir SQL para cada petición de nuestro departamento de negocio. 
Nuestro trabajo se reducirá a la creación y mantenimiento de los metadatos y sobre todo a preparar las tablas de hechos para que el acceso sea lo más
eficiente posible.

## Siguientes pasos

La generación de SQL es correcta en casi todos los casos pero aún le quedan algunas cosas complicadas como por ejemplo las jerarquías anidadas. Habréis observado
que no he intentado hacer ninguna consulta sobre la ciudad en la que tiene su sede el proveedor de un producto. Aún no se generan las consultas para
jerarquías doblemente anidadas, me di cuenta ayer, supongo que en estos días terminaré con ello.

La parte de interface de usuario, como habréis notado, aún es mejorable y permite hacer cosas como jerarquías de dimensiones recursivas que no deberían
existir.

Por último, los informes están pensados para que se puedan alimentar de diferentes orígenes de datos. Por ejemplo, si sólo queremos la suma de ventas entre
fechas, no parece tener sentido ir sobre la tabla *FactSales* que puede tener millones de registros, parece más lógico crear una tabla *FactGroupedSales*
con los datos previamente sumados y si el usuario no escoge ninguna dimensión ejecutar nuestra consulta sobre esta segunda tabla.

En algunos casos, también nos puede interesar hacer filtros por datos adicionales, por ejemplo un periodo adicional, esto no está contemplado en esta versión.

Ahora mismo, **BauDbStudio** no nos permite agregar datos sobre el mismo campo, es decir, no nos permite ver la suma y la media de precios a la vez, ni por
supuesto consultas más complejas como un porcentaje sobre el total o una fórmula sobre los registros. En realidad es un problema del interface de usuario, no
del generador de SQL e iré añadiendo esta opción en el futuro.

Todos estos puntos los iré desarrollando pero el resto es plenamente funcional.

Como siempre, tenéis la aplicación en [GitHub](https://github.com/jbautistam/BauDbStudio/releases) por si queréis hacer alguna prueba.
