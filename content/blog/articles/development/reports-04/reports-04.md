+++
title = "Generación de informes - 4"
date = "2024-03-18"
description = "Generación de informes: parte 4"
thumbnail = "/images/noimage.jpg"
tags = [ "Development", "Reports" ]
+++

Tras un pequeño parón sigamos con la [serie](/blog/articles/development/reporting/reports-01) de artículos sobre la librería de generación de informes. 

En esta ocasión, vamos a ver una serie de ejemplos de cómo funciona la librería y cómo se definen sus metadatos.

Voy a utilizar [BauDbStudio](https://github.com/jbautistam/BauDbStudio) para capturar las pantallas y explicar las funcionalidades de la librería. 
Espero que así quede más claro pero recordemos que la librería es independiente. No hace falta utilizar esta aplicación para definir sus parámetros,
simplemente me va a facilitar las cosas.

## Base de datos

Si queremos informes, vamos a necesitar una base de datos así que me he creado una base de datos básica de ventas con empleados, clientes, tiendas,
productos, ventas y stocks.

Así, el diagrama de empleados sería el siguiente:

![Empleados](/blog/articles/development/reports-04/images/erd-employees.png)

Aparte de los datos del cliente, la tabla `EmployeeSalaries` es una tabla de hechos con los sueldos de los empleados.

El diagrama de tiendas es el siguiente:

![Tiendas](/blog/articles/development/reports-04/images/erd-shops.png)

La tablas de stock por tienda son las siguientes:

![Stock](/blog/articles/development/reports-04/images/erd-stocks.png)

Y las tablas de ventas son las siguientes:

![Ventas](/blog/articles/development/reports-04/images/erd-sales.png)

Por último, la entidad de productos tiene este diagrama:

![Productos](/blog/articles/development/reports-04/images/erd-products.png)

En esta entidad tenemos la entidad raíz de los productos que agrupa productos similares y que tienen asociados 
productos.

Los productos raíz tienen definida una categoría mientras que los productos tienen asociada talla. Así, nuestro
producto raíz sería una camisa (por ejemplo) mientras que el producto sería la talla L de esa camisa. Por supuesto,
las ventas y el stock está asociadas a productos, no a productos raíz.

Pero aquí me gustaría resaltar las entidades de etiquetas (`Tags`, `TagValues` y `EntityTagValues`).

El problema que resuelven estas tablas son las infinitas características que puede tener un producto: color, temporada,
familia, marca, público objetivo... Si lo tratáramos como campos, es posible que tuviéramos que añadir campos de
`ColorId`, `SeasonId`, `BrandId`, `FamilyId`... a nuestra tabla de productos y por supuesto, cada vez que se nos
ocurriera una categoría deberíamos añadir nuevos campos.

Tenemos un problema adicional y es que no todos los productos tienen las mismas características: si vendemos camisas
el color es importante pero si además vendemos barras de pan, el color deja de tener importancia. 

Podríamos establecer un número máximo de categorías con un número máximo de campos fijos y distinguirlos por un tipo.
Por ejemplo, los productos de tipo 1 tienen en el campo 1 el color y en el campo 2 el tamaño mientras que los productos
de tipo 2 tienen en el campo 1 la marca y en el campo 2 la temporada. Es una opción pero antes o después vamos a quedarnos
cortos ¿cuál es el número máximo 10? ¿20? ¿5?.

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

Parece muy complicado para realizar una consulta de este tipo, pero nos facilitará mucho la generación de informes más tarde.

Hemos pasado de puntillas por el campo `EntityId` de la tabla `Tags`. Este campo nos indica la entidad a la que se asocia la etiqueta.
Ya que tenemos una definición de etiquetas, podemos utilizarlas también para definir clasificaciones adicionales en tiendas, productos, ...

Podríamos tener diferentes tablas del tipo `TagsShops` o `TagsProducts` pero me parecía más cómodo así.

En el ejemplo, sólo he definido etiquetas para los códigos de producto raíz por no liarme demasiado.

Pero entonces, si ya tenemos etiquetas ¿para qué queremos un campo `ClassificationTypeId` en la tabla de productos raíz?

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

Por cada origen de datos (tabla), podemos definirle algunos datos adicionales que vamos a utilizar en el futuro. Por ejemplo, en la 
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
ciudades. Tal como tenemos nuestros datos, las tiendas y el resto de datos se relacionan con la ciudado, por tanto nos interesa tener 
las dimensiones al revés, es decir, una ciudad pertenece a una región que pertenece a un país.

Por tanto, nuestro siguiente paso es definir las dimensiones relacionadas. Por ejemplo, la dimensión `Cities` se relaciona
con la dimensión `States` por el campo `CountryId`:

![Relación Countries con States](/blog/articles/development/reports-04/images/logical-06.png)

y, del mismo modo, la dimensión `Cities` se relaciona con `States` por el campo `StateId`. Ahora ya podemos ver el árbol de dimensiones / subdimensiones
con esos datos:

![Arbol de dimensiones](/blog/articles/development/reports-04/images/logical-07.png)

Vamos a hacer lo mismo con la dimensión `Shops` sobre la tabla de tiendas. Esta dimensión está relacionada con otra dimensión nueva: los tipos de tienda y
con la dimensión que acabamos de crear con ciudades:

## Creación de informes