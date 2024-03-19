+++
title = "Generación de informes - 4"
date = "2024-03-18"
description = "Generación de informes: parte 4"
thumbnail = "/images/noimage.jpg"
tags = [ "Development", "Reports" ]
+++

Tras un pequeño parón sigamos con la [serie de artículos](/blog/articles/development/reporting/reports-01) sobre la librería de generación de informes. 

En esta ocasión, vamos a ver una serie de ejemplos de cómo funciona la librería y cómo se definen sus metadatos.

Voy a utilizar [BauDbStudio](https://github.com/jbautistam/BauDbStudio) para capturar las pantallas y explicar las funcionalidades de la librería. 

Espero que así quede más claro pero recordemos que la librería es independiente. No hace falta utilizar esta aplicación para definir sus parámetros,
simplemente me va a facilitar (espero) la explicación de los diferentes aspectos que necesitamos en la librería.

## Base de datos

Si queremos informes, vamos a necesitar una base de datos: me he creado una de ventas con empleados, clientes, tiendas,
productos y stocks.

En esta base de datos, el diagrama de empleados sería el siguiente:

![Empleados](/blog/articles/development/reports-04/images/erd-employees.png)

Aparte de los datos del empleado, la tabla `EmployeeSalaries` es una tabla de hechos con los sueldos de los empleados.

El diagrama de tiendas es el siguiente:

![Tiendas](/blog/articles/development/reports-04/images/erd-shops.png)

La tablas de stock por tienda son las siguientes:

![Stock](/blog/articles/development/reports-04/images/erd-stocks.png)

Y las tablas de ventas son las siguientes:

![Ventas](/blog/articles/development/reports-04/images/erd-sales.png)

Por último, la entidad de productos tiene este diagrama:

![Productos](/blog/articles/development/reports-04/images/erd-products.png)

En esta entidad tenemos la entidad raíz de los productos que agrupa productos similares y que tienen asociados 
productos o artículos.

Los productos raíz tienen definida una categoría mientras que los productos tienen asociada talla. Así, nuestro
producto raíz sería una camisa (por ejemplo) mientras que el producto sería la talla L de esa camisa. Por supuesto,
las ventas y el stock están asociados a productos, no a productos raíz.

En este punto, me gustaría resaltar las entidades de etiquetas (`Tags`, `TagValues` y `EntityTagValues`).

El problema que resuelven estas tablas son las infinitas características que puede tener un producto: color, temporada,
familia, marca, público objetivo... Si lo tratáramos como campos, es posible que tengamos que añadir campos de
`ColorId`, `SeasonId`, `BrandId`, `FamilyId`... a nuestra tabla de productos y por supuesto, cada vez que se nos
ocurra una categoría nueva deberíamos añadir nuevos campos.

Tenemos un problema adicional y es que no todos los productos tienen las mismas características: si vendemos camisas
el color es importante pero si además vendemos barras de pan, el color no es algo vital. 

Podríamos establecer un número máximo de categorías con un número máximo de campos fijos y distinguirlos por tipo.
Por ejemplo, los productos de tipo 1 tienen en el campo 1 el color y en el campo 2 el tamaño mientras que los productos
de tipo 2 tienen en el campo 1 la marca y en el campo 2 la temporada. Es una opción pero antes o después vamos a quedarnos
cortos ¿cuál es el número máximo 10 campos adicionales? ¿20? ¿5?.

Para evitar este problema es para lo que tenemos las etiquetas. Podemos tener diferentes tipos de etiquetas, por ejemplo
en la base de datos he definido cuatro etiquetas para cuatro categorías: color, temporada, marca y grupo de edad objetivo:

![Etiquetas](/blog/articles/development/reports-04/images/tags.png)

y en `TagValues` se indican los posibles valores:

![Valores de etiquetas](/blog/articles/development/reports-04/images/tag-values.png)

Por su parte, la tabla, `EntityTagValues` relaciona los productos raíz con los valores de las etiquetas:

![Valores de los productos raíz](/blog/articles/development/reports-04/images/entity-tag-values.png)

Así, podemos consultar por ejemplo el color de los productos raíz:

```sql
SELECT ProductRoots.ProductRootId, ProductRoots.Name, Tags.Name AS Type, TagValues.Name AS Value
	FROM Dim.ProductRoots INNER JOIN Dim.EntityTagValues
		ON ProductRoots.ProductRootId = EntityTagValues.EntityValueId
			AND EntityTagValues.EntityId = 2
	INNER JOIN Dim.TagValues
		ON EntityTagValues.TagValueId = TagValues.TagValueId
	INNER JOIN Dim.Tags
		ON TagValues.TagId = Tags.TagId
```

con este resultado:

![Colores de los productos raíz](/blog/articles/development/reports-04/images/product-root-colours.png)

Este tipo de consultas parecen demasiado complicadas, pero nos facilitarán mucho la generación de informes más tarde.

Hemos pasado de puntillas por el campo `EntityId` de la tabla `Tags`. Este campo nos indica la entidad a la que se asocia la etiqueta.
Ya que tenemos una definición de etiquetas, podemos utilizarlas también para definir clasificaciones adicionales en tiendas, productos, ...

Podríamos tener diferentes tablas del tipo `TagsShops` o `TagsProducts` pero me parecía más cómodo así.

En el ejemplo, sólo he definido etiquetas para los códigos de producto raíz por no liarme demasiado.

Pero entonces: si ya tenemos etiquetas ¿para qué queremos un campo `ClassificationTypeId` en la tabla de productos raíz?

En primer lugar: por rendimiento, si sabemos que siempre vamos a tener ese campo en todos los productos raíz va a ser más
rápido obtenerlo de la tabla que buscando por las tablas de etiquetas. 

La segunda razón es más prosaica, simplemente quería tener ambas opciones en la base de datos.

## Definición del modelo físico

Una vez tenemos una base de datos de pruebas, vamos a definir el modelo físico. 

Como decía al principio, voy a obtener un archivo XML del esquema de la base de datos con **BauDbStudio**:

![Generando el esquema físico](/blog/articles/development/reports-04/images/physical-schema.png)

esta opción genera un archivo más o menos de este tipo:

```xml
<?xml version='1.0' encoding='utf-8'?>
<Schema>
	<Table Schema = "Dim"  Catalog = "SalesSample"  Name = "Calendar" >
		<Field Name = "CalendarId"  Type = "Integer"  DbType = "int"  IsKey = "yes"  Length = "0"  Required = "yes"  Position = "1"  Identity = "yes" />
		<Field Name = "Date"  Type = "Date"  DbType = "date"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "2"  Identity = "no" />
		<Field Name = "Year"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "3"  Identity = "no" />
		<Field Name = "Month"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "4"  Identity = "no" />
		<Field Name = "Day"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "5"  Identity = "no" />
		<Field Name = "Week"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "6"  Identity = "no" />
		<Field Name = "YearWeek"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "7"  Identity = "no" />
		<Field Name = "YearMonth"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "8"  Identity = "no" />
		<Field Name = "WeekDay"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "9"  Identity = "no" />
		<Field Name = "NaturalYear"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "10"  Identity = "no" />
		<Field Name = "NaturalWeek"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "11"  Identity = "no" />
		<Field Name = "LastWeekDay"  Type = "Boolean"  DbType = "bit"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "12"  Identity = "no" />
		<Field Name = "LastMonthDay"  Type = "Boolean"  DbType = "bit"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "13"  Identity = "no" />
		<Field Name = "YearWeekIndex"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "14"  Identity = "no" />
		<Field Name = "YearMonthIndex"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "15"  Identity = "no" />
		<Constraint Schema = "Dim"  Catalog = "SalesSample"  Name = "PK_Calendar"  Table = "Calendar"  Field = "CalendarId"  Type = "PrimaryKey"  Position = "1" />
	</Table>
	<Table Schema = "Dim"  Catalog = "SalesSample"  Name = "Cities" >
		<Field Name = "CityId"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "1"  Identity = "no" />
		<Field Name = "StateId"  Type = "Integer"  DbType = "int"  IsKey = "no"  Length = "0"  Required = "yes"  Position = "2"  Identity = "no" />
		<Field Name = "Name"  Type = "String"  DbType = "nvarchar"  IsKey = "no"  Length = "200"  Required = "yes"  Position = "3"  Identity = "no" />
	</Table>
	...
</Schema>
```

con las definiciones de tablas y campos de la base de datos. Este esquema será el que utilizaremos para definir el modelo lógico.

## Definición del modelo lógico

Como decía en el artículo anterior, nuestro modelo lógico define los metadatos, dimensiones y los diferentes orígenes de datos que vamos a utilizar al
ver nuestros informes.

Para ello, voy a utilizar de nuevo **BauDbStudio**. 

Lo primero, es crear un nuevo proyecto de informes con el esquema que acabamos de crear en el paso anterior:

![Creando un proyecto de informes](/blog/articles/development/reports-04/images/logical-01.png)

Esta importación, ya nos rellena los orígenes de datos a partir del esquema:

![Orígenes de datos](/blog/articles/development/reports-04/images/logical-02.png)

Por cada origen de datos (tabla), podemos añadir datos que vamos a utilizar en el futuro. Por ejemplo, en la 
tabla de productos, he definido los alias de un par de campos y he marcado los campos `ProductId`, `ProductRootId` y `SizeId` como
invisibles para que el usuario no los vea cuando seleccione los productos en la interface de usuario:

![Orígenes de datos: productos](/blog/articles/development/reports-04/images/logical-03.png)

Aparte de modificar los orígenes de datos, podemos añadir dimensiones a partir de un origen de datos. Por ejemplo, agreguemos
una dimensión para el país a partir del origen de datos `Countries`:

![Dimensiones: countries](/blog/articles/development/reports-04/images/logical-04.png)

En este caso simplemente le cambiamos el nombre:

![Dimensión: Countries](/blog/articles/development/reports-04/images/logical-05.png)

y definimos del mismo modo las dimensiones de regiones y ciudades que ya nos aparecen en nuestro esquema lógico:

![Dimensiones](/blog/articles/development/reports-04/images/logical-06.png)

Pero lo que nos interesa es ver las relaciones internas, es decir, un país tiene varias regiones y cada región puede tener diferentes
ciudades. 

Tal como tenemos nuestros datos, las tiendas y el resto de tablas se relacionan con la ciudades, por tanto nos interesa más tener 
las dimensiones al revés: una ciudad pertenece a una región que pertenece a un país.

Así, nuestro siguiente paso es definir estas relaciones entre dimensiones: la dimensión `Cities` se relaciona
con la dimensión `States` por el campo `CountryId`:

![Relación Countries con States](/blog/articles/development/reports-04/images/logical-06.png)

y, del mismo modo, la dimensión `States` se relaciona con `Countries` por el campo `CountryId`. Ahora ya podemos ver el árbol de dimensiones / subdimensiones
con esos datos:

![Arbol de dimensiones](/blog/articles/development/reports-04/images/logical-07.png)

Vamos a hacer lo mismo con la dimensión `Shops` sobre la tabla de tiendas. Esta dimensión está relacionada con otra dimensión nueva: los tipos de tienda y
con la dimensión que acabamos de crear con ciudades:

![Dimensión de tienda](/blog/articles/development/reports-04/images/logical-09.png)

Como vemos, antes de generar nuestros informes, tenemos que preparar el esquema lógico. Todos son iguales así que voy a dejar el ejemplo aquí e indicaré
en el futuro aquellos pasos que encuentre interesantes.

El caso es que **BauDbStudio** graba todos estas definiciones en otro archivo XML que deja junto al archivo del esquema físico:

![Esquema lógico](/blog/articles/development/reports-04/images/logical-10.png)

Este archivo tiene más o menos este aspecto:

```xml
<?xml version='1.0' encoding='utf-8'?>
<Dashboard>
	<Name>Sql Server - SalesSamples.Schema</Name>
	<Description/>
	<DataSourceTable Id = "[Dim].[Cities]"  Schema = "Dim"  Table = "Cities"  IsView = "no" >
		<Column Alias = "CityId"  SourceId = "CityId"  IsPrimaryKey = "no"  Visible = "yes"  Type = "Integer"  Required = "yes" />
		<Column Alias = "StateId"  SourceId = "StateId"  IsPrimaryKey = "no"  Visible = "yes"  Type = "Integer"  Required = "yes" />
		<Column Alias = "Name"  SourceId = "Name"  IsPrimaryKey = "no"  Visible = "yes"  Type = "String"  Required = "yes" />
	</DataSourceTable>
	
	...
	
	<DataSourceTable Id = "[Dim].[Products]"  Schema = "Dim"  Table = "Products"  IsView = "no" >
		<Column Alias = "ProductCode"  SourceId = "Code"  IsPrimaryKey = "no"  Visible = "yes"  Type = "String"  Required = "yes" />
		<Column Alias = "Cost"  SourceId = "Cost"  IsPrimaryKey = "no"  Visible = "yes"  Type = "Decimal"  Required = "yes" />
		<Column Alias = "ProductName"  SourceId = "Name"  IsPrimaryKey = "no"  Visible = "yes"  Type = "String"  Required = "yes" />
		<Column Alias = "Price"  SourceId = "Price"  IsPrimaryKey = "no"  Visible = "yes"  Type = "Decimal"  Required = "yes" />
		<Column Alias = "ProductId"  SourceId = "ProductId"  IsPrimaryKey = "yes"  Visible = "no"  Type = "Integer"  Required = "yes" />
		<Column Alias = "ProductRootId"  SourceId = "ProductRootId"  IsPrimaryKey = "no"  Visible = "no"  Type = "Integer"  Required = "yes" />
		<Column Alias = "SizeId"  SourceId = "SizeId"  IsPrimaryKey = "no"  Visible = "no"  Type = "Integer"  Required = "yes" />
	</DataSourceTable>
	
	...
	
	<Dimension Id = "Countries"  SourceId = "[Dim].[Countries]" />
	<Dimension Id = "States"  SourceId = "[Dim].[States]">
		<Relation SourceId = "Countries" >
			<ForeignKey Column = "CountryId"  DimensionColumn = "CountryId" />
		</Relation>
	</Dimension>
	<Dimension Id = "Cities"  SourceId = "[Dim].[Cities]" >
		<Relation SourceId = "States" >
			<ForeignKey Column = "StateId"  DimensionColumn = "StateId" />
		</Relation>
	</Dimension>
	
	...
	
	<Dimension Id = "Shops"  SourceId = "[Dim].[Shops]" >
		<Description/>
		<Relation SourceId = "ShopTypes" >
			<ForeignKey Column = "ShopTypeId"  DimensionColumn = "ShopTypeId" />
		</Relation>
		<Relation SourceId = "Cities" >
			<ForeignKey Column = "CityId"  DimensionColumn = "CityId" />
		</Relation>
	</Dimension>
</Dashboard>
```

En el archivo vemos los orígenes de datos con la información de la tabla incluyendo alias y visibilidad de los campos. En las dimensiones se
recogen el origen de datos y las relaciones con otras dimensiones.

Voy a repetirlo una vez más, este es simplemente el archivo XML de salida de **BauDbStudio**, vamos a poder utilizar la librería de informes
sin necesidad de estos archivos siempre que especifiquemos de alguna forma esta estructura.

## Creación de informes

Una vez definido el esquema lógico, podemos crear el informe.

La librería va a generar comandos SQL que obtengan ciertos datos, pero para indicar cómo se debe crear este comando SQL necesitamos saber las dimensiones que
vamos a utilizar, las expresiones de salida y los bloques de código que nos permiten generar esa cadena.

Quizá la forma más fácil de verlo es con un ejemplo en XML:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Report>
	<Name>Sales report</Name>
	
	<Parameter Name = "StartDate" Type = "Date" />
	<Parameter Name = "EndDate" Type = "Date" />

	<Dimension Name = "Shops" />
	<Dimension Name = "Products" />
	<Dimension Name = "Calendar" />

	<Requests>
		<Dimension Name = "Shops">
			<Field Name = "Name;ErpCode" />
		</Dimension>
		<Dimension Name = "Products">
			<Field Name = "ProductCode" />
		</Dimension>
		<Dimension Name = "Calendar" />
	</Requests>

	<Expression Name = "Quantity;SellingPrice;PurchasePrice;Taxes" />

	<Blocks>
		<With Name = "Main">
			<IfRequest Dimension = "Products">
				<Cte Name = "ProductsCte" Dimension = "Products" />
			</IfRequest>

			<IfRequest Dimension = "Shops">
				<Cte Name = "ShopsCte" Dimension = "Shops" />
			</IfRequest>

			<Cte Name = "CalendarCte" Dimension = "Calendar">
				<Filter>
					<![CDATA[
	 					[Date] BETWEEN @StartDate AND @EndDate
	 				]]>
				</Filter>
			</Cte>

			<Query>
				<![CDATA[
					SELECT {{
								Fields
									-Dimension: Shops 
										--Table: ShopsCte
									-Dimension: Products 
										--Table: ProductsCte
									-Dimension: Calendar 
										--Table: CalendarCte
									-WithComma
	                		}}
	                		SUM(Sales.Quantity) AS Quantity, SUM(Sales.SellingPrice) AS SellingPrice, 
	                		SUM(Sales.PurchasePrice) AS PurchasePrice, SUM(Sales.Taxes) AS Taxes
	                	FROM Fact.Sales
	                	{{
	                		InnerJoin 
	                			-Dimension: Shops 
	                				--Table: ShopsCte
								-Table: Sales
								-On: ShopId-ShopId
	                	}}
	                	{{
	                		InnerJoin 
	                			-Dimension: Products 
	                				--Table: ProductsCte
								-Table: Sales
								-On: ProductId-ProductId
	                	}}
	                	{{
	                		InnerJoin 
	                			-Dimension: Calendar 
	                				--Table: CalendarCte
								-Table: Sales
								-On: CalendarId-CalendarId
								-Required
	                	}}
						{{
	                		GroupBy
	                			-Dimension: Shops
									--Table: ShopsCte
		            			-Dimension: Products
									--Table: ProductsCte
								-Dimension: Calendar
									--Table: CalendarCte
		            	}}
						{{
	                		OrderBy
	                			-Dimension: Shops
									--Table: ShopsCte
		            			-Dimension: Products
									--Table: ProductsCte
								-Dimension: Calendar
									--Table: CalendarCte									
		            	}}
						{{Pagination}}
				]]>
			</Query>
		</With>
	</Blocks>
</Report>
```

Aparte del nombre del archivo (*Sales report*), se definen parámetros (las etiquetas `Parameter` del archivo), las dimensiones que se van a utilizar 
(con las etiquetas `Dimension`), las expresiones (`Expression`) de salida y los bloques con los que se van a formar la SQL (el nodo `Blocks`).

Vamos a generar la SQL a partir de bloques `With` (que definimos en un nodo) y dentro de esos bloques se añaden las dimensiones y las consultas finales. 

En este caso, los nodos `IfRequest` indican que se incluyan las SQL apropiadas para una dimensión si el usuario ha solicitado algún campo (por ejemplo, si el
usuario solicita un `ErpCode` de la tienda, se añadirá la CTE de tiendas pero si no se solicita ningún campo de tiendas, no se añade ninguna CTE).

El bloque `Query` define la SQL final que mezcla nuestras CTE.

Como se puede ver, hay mucho que explicar en ese XML, por eso dejaremos toda esa explicación para los siguientes artículos.

## Ejecución de informe

Pero aunque no veamos el contenido sí podemos hacernos una idea de cómo funcionaría la librería utilizando nuevamente **BauDbStudio**.

En primer lugar, hemos añadido el XML del informe a nuestro esquema lógico:

![Informe: 01](/blog/articles/development/reports-04/images/report-01.png)

como vemos, el informe tiene tres dimensiones asociadas: *Shops*, *Products* y *Calendar*. Es decir, por ahora, podemos filtrar las ventas
por estas dimensiones. No se ha definido que se pueda filtrar por empleados aunque lo veremos en el futuro.

Si pulsamos sobre la opción *Consulta* del informe:

![Informe: 02](/blog/articles/development/reports-04/images/report-02.png)

veremos la ventana de selección y filtro:

![Informe: 03](/blog/articles/development/reports-04/images/report-03.png)

**Nota:** en la dimensión de tiendas aparecen campos como `CityId` porque no lo he señalado aún como invisible al crear la dimensión aunque
no tiene sentido que el usuario pueda seleccionarlo.

Aquí podemos seleccionar los campos que queremos: por ejemplo, la fecha, los datos de tienda y los datos de productos para todas las expresiones del informe
y pulsar el botón de ejecutar para que la librería genere la consulta SQL y obtenga el resultado:

![Informe: 04](/blog/articles/development/reports-04/images/report-04.png)

Por supuesto, también podemos decirle que sólo saque los datos de fecha y tiendas sin los productos simplemente deseleccionado los campos de producto:

![Informe: 05](/blog/articles/development/reports-04/images/report-05.png)

o filtrar para una única tienda, primero añadiendo la condición:

![Informe: 06](/blog/articles/development/reports-04/images/report-06.png)

y después ejecutando la consulta:

![Informe: 07](/blog/articles/development/reports-04/images/report-07.png)

en la sentencia SQL, en la CTE de tiendas, se ha añadido el filtro deseado.

Por supuesto, podríamos añadir los campos de país o filtrar por producto para obtener la suma de esos productos en una tienda o añadir el
campo de trimestre de calendario en lugar de la fecha para acumular por trimestre... pero al menos hemos demostrado que funciona la abstracción de la que
hablábamos en el [segundo artículo](/blog/articles/development/reports-02/reports-02).

## Conclusión

Hemos visto cómo podemos crear un informe genérico y los resultados de la librería utilizando **BauDbStudio**, en los siguientes artículos hablaremos primero
de creación de pruebas y después las diferentes instrucciones que conformarán los informes.
