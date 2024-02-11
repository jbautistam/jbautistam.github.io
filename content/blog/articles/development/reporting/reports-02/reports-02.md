+++
title = "Generación de informes - 2"
date = "2024-02-10"
description = "Generación de informes: parte 2"
thumbnail = "/images/noimage.jpg"
tags = [ "Development", "Reports" ]
+++

Cerramos el [artículo anterior](/blog/articles/development/reporting/reports-01/reports-01) con varias preguntas sobre informes que nos gustaría generar:

1. Las ventas de los vendedores entre dos fechas.
2. Las compras de los clientes entre dos fechas.
3. Las ventas en una tienda entre dos fechas.
4. Las ventas de los vendedores por cada tienda entre dos fechas.
5. Las compras de los clientes por cada tienda entre dos fechas.
6. Las compras de los clientes por cada vendedor en cada tienda entre dos fechas.
7. Las compras de los clientes por cada vendedor en cada tienda de Madrid entre dos fechas.

Por supuesto, podríamos generarlos siguiendo la estructura mostrada en el primer artículo: añadimos
un registro por cada uno de los informes con una SQL asociada y una serie de filtros. 

No estaría mal pero ya que hablamos de abstracciones quizá, si profundizamos algo más, obtengamos una respuesta
más sencilla. O posiblemente una abstracción mucho más compleja pero también más flexible.

Fijémonos en los siete informes anteriores: tienen algo en común, todos hablan de ventas (en realidad,
las compras no son más que una venta a un comprador).

Si definimos los registros de nuestra tabla de ventas tendremos a primera vista los siguientes campos:

| Sales      |
|------------|
| Store      |
| Seller     |
| Customer   |
| Product	 |
| Date		 |
| Quantity	 |
| Price 	 |
| Cost       |

Podríamos tener muchos más campos, o incluso quitar los campos de `Price` y `Cost` y mirar en el histórico
de precios por producto, pero vamos a dejarlo así para que sea más sencillo.

De esos campos, al normalizar la tabla: la tienda, el vendedor, el comprador y el producto son realmente identificadores relacionados con otras tablas.
De hecho, la fecha puede ser también un identificador en otra tabla de calendario.

Los campos que nos quedan, la cantidad, el precio, y el coste son los datos reales de esa tabla: los que podemos
contabilizar y tratar. El resto nos sirve para filtrar y para añadir información.

Si vamos a un diagrama de la base de datos:

![Diagrama de entidad relación](/blog/articles/development/reporting/images/erd-diagram-1.png)

Podríamos distinguir entre dos tipos de tablas o entidades:

1. Las [tabla de hechos](https://es.wikipedia.org/wiki/Tabla_de_hechos) en las que almacenamos los valores de negocio 
y las relaciones con las dimensiones que la definen.
2. Las [tabla de dimensiones](https://es.wikipedia.org/wiki/Tabla_de_dimensi%C3%B3n) donde se almacenan atributos o 
campos utilizados para restringir las consultas de los datos almacenados en las tablas de hechos.

En nuestro caso, la tabla de hechos es `Sales` mientras que el resto de las entidades son dimensiones.

Por supuesto, no tenemos porqué quedarnos en una estructura tan sencilla. Una tabla de dimensiones, también puede
tener dimensiones asociadas. Por ejemplo, a las tiendas le podríamos añadir la ciudad o
la categoría de la tienda dentro de la empresa. Al producto le podríamos añadir una tipología o etiquetas para clasificarlo.

![Diagrama de entidad relación](/blog/articles/development/reporting/images/erd-diagram-2.png)

Y no estamos obligados tampoco a tener una única tabla de hechos, podemos tener otras tablas relacionadas o no con
las mismas entidades, por ejemplo, si añadimos a nuestro diagrama una tabla de stocks:

![Diagrama de entidad relación](/blog/articles/development/reporting/images/erd-diagram-3.png)

Vemos que la tabla de hechos `Stocks` se relaciona con las dimensiones `Products` y `Stores` mientras que la tabla de
hechos `Sales` se relaciona además con las dimensiones `Customers`, `Sellers` y `Calendar`.

**Nota:** he quitado de este último diagrama las *subdimensiones* para que resulte más claro pero siguen estando ahí.

## ¿Por qué distinguir entre dimensiones y hechos?

Diferenciar entre distintos tipos de tablas en nuestra base de datos, nos permite separar conceptos. 

Todo aquello que definamos como dimensión mantendrá atributos y se relacionará única y exclusivamente con otras dimensiones
y con las tablas de hechos, mientras que aquello que definamos como hecho, mantendrá valores de negocio y única y exclusivamente 
se relacionará con dimensiones (normalmente, no con *subdimensiones* aunque puede llegar a darse el caso).

Esto nos permite expresar los informes que hemos esbozado al principio del artículo en relación con las dimensiones
y hechos que tenemos en la base de datos.

Por ejemplo:

1. **Las ventas de los vendedores entre dos fechas:** se traduce en, la suma de `Quantity` (o `Price` o `Cost` o la diferencia entre `Price` y `Cost` o todos ellos ... 
ya nos entendemos) del hecho `Sales` relacionada con las dimensiones `Sellers` y `Calendar` filtrado entre dos fechas.
2. **Las compras de los clientes entre dos fechas:** se traduce en, la suma de `Quantity` del hecho `Sales` 
relacionada con la dimensiones `Customers` y `Calendar` filtrado entre dos fechas.
3. **Las ventas en una tienda entre dos fechas:**  se traduce en, la suma de `Quantity`
del hecho `Sales` relacionada con las dimensiones `Stores` y `Calendar` filtrado entre dos fechas.

Por no extendernos, si vamos directamente al último informe:

7. **Las compras de los clientes por cada vendedor en cada tienda de Madrid entre dos fechas:** se traduce en, la suma de `Quantity` 
del hecho `Sales` relacionada con las dimensiones `Sellers`, `Stores` (filtrada por la entidad `Cities`) 
y `Calendar` filtrado entre dos fechas.

En este caso ya podemos ver una abstracción: dimensiones, hechos y las relaciones entre ellos nos permiten definir prácticamente
cualquier informe.

Además, pensando en un informe no solicitado aún ¿cuáles serían las ventas totales? La
suma de la entidad de hechos `Sales` sin ninguna dimensión.

## SQL: nuestro arma secreta

Una vez definidos los informes que necesitamos y la estructura de las tablas o el diagrama que pretendemos utilizar ¿cómo
podemos escribir una herramienta que nos resuelva todos estos informes?

Siguiendo con el artículo anterior, parece que la respuesta obligada es **SQL** pero cabe preguntarse si podemos hacerlo utilizando
clases y ORM. Posiblemente sí, pero volvemos a tener el problema que describimos en el artículo anterior: para mí, intentar
definir todos los posibles informes a partir de clases se acaba convirtiendo en un infierno bastante complicado de manejar.

Por ahora, confiemos en que SQL nos pueda ayudar a solucionar la papeleta.

Vamos a hacer una prueba con un informe sencillo: las ventas totales, es decir, una consulta simple sobre la tabla de hechos (a
partir de ahora voy a escribir las consultas de memoria, sin probar, que a nadie se le ocurra hacer esto en producción):

```sql
SELECT SUM(Quantity) AS Quantity, SUM(Price) AS Price
	FROM Fact.Sales
```

**Nota:** voy a prefijar las tablas con su nombre de esquema: `Dim` será el esquema de las dimensiones
mientras que `Fact` será el esquema de las tablas de hechos. No es estrictamente necesario pero es lo que se suele hacer
para mantenerlos separados y que sean fácilmente localizables.

Ahora vamos a otro informe sencillo, las ventas totales entre dos fechas:

```sql
SELECT SUM(Quantity) AS Quantity, SUM(Price) AS Price
	FROM Fact.Sales
	WHERE Date BETWEEN '2022-01-01' AND '2022-12-31'
```

Aquí hemos hecho algo de trampa. Teníamos una dimensión de calendario así pues vamos a utilizarla:

```sql
SELECT SUM(Sales.Quantity) AS Quantity, SUM(Sales.Price) AS Price
	FROM Fact.Sales INNER JOIN Dim.Calendar
		ON Sales.Date = Calendar.Date
	WHERE Calendar.Date BETWEEN '2022-01-01' AND '2022-12-31'
```

En este caso estamos enlazando por `Date` que en realidad podríamos filtrar directamente. Lo lógico es que
la tabla `Calendar` tuviera un `Id` numérico y enlazásemos por ese valor. Hasta hace poco, solía ser más eficiente
relacionar tablas por valores numéricos y aunque en la actualidad no haya demasiada diferencia en cuanto a velocidad,
posiblemente sigue existiendo esa diferencia en cuanto a tamaño de la tabla.

Es decir, quedaría algo así:

```sql
SELECT SUM(Sales.Quantity) AS Quantity, SUM(Sales.Price) AS Price
	FROM Fact.Sales INNER JOIN Dim.Calendar
		ON Sales.CalendarId = Calendar.Id
	WHERE Calendar.Date BETWEEN '2022-01-01' AND '2022-12-31'
```

Y si queremos ver las ventas entre dos fechas agrupadas por día, podemos hacer esto:

```sql
SELECT Calendar.Date, SUM(Sales.Quantity) AS Quantity, SUM(Sales.Price) AS Price
	FROM Fact.Sales INNER JOIN Dim.Calendar
		ON Sales.CalendarId = Calendar.Id
	WHERE Calendar.Date BETWEEN '2022-01-01' AND '2022-12-31'
	GROUP BY Calendar.Date
```

Vamos a añadir algo más, vamos a ver las ventas entre dos fechas por tienda agrupada por nombre de tienda y fecha:

```sql
SELECT Stores.Name, Calendar.Date, 
	   SUM(Sales.Quantity) AS Quantity, SUM(Sales.Price) AS Price
	FROM Fact.Sales INNER JOIN Dim.Calendar
		ON Sales.CalendarId = Calendar.Id
	INNER JOIN Dim.Stores
		ON Sales.StoreId = Stores.Id
	WHERE Calendar.Date BETWEEN '2022-01-01' AND '2022-12-31'
	GROUP BY Stores.Name, Calendar.Date
```

Y ahora le añadimos el producto:

```sql
SELECT Stores.Name, Products.Name, Calendar.Date, 
	   SUM(Sales.Quantity) AS Quantity, SUM(Sales.Price) AS Price
	FROM Fact.Sales INNER JOIN Dim.Calendar
		ON Sales.CalendarId = Calendar.Id
	INNER JOIN Dim.Stores
		ON Sales.StoreId = Stores.Id
	INNER JOIN Dim.Products
		ON Sales.ProductId = Products.Id
	WHERE Calendar.Date BETWEEN '2022-01-01' AND '2022-12-31'
	GROUP BY Stores.Name, Products.Name, Calendar.Date
```

Ahora sólo para las tiendas de Madrid:

```sql
SELECT Stores.Name, Products.Name, Calendar.Date, 
	   SUM(Sales.Quantity) AS Quantity, SUM(Sales.Price) AS Price
	FROM Fact.Sales INNER JOIN Dim.Calendar
		ON Sales.CalendarId = Calendar.Id
	INNER JOIN Dim.Stores
		ON Sales.StoreId = Stores.Id
	INNER JOIN Dim.Cities
		ON Stores.CityId = Cities.Id
	INNER JOIN Dim.Products
		ON Sales.ProductId = Products.Id
	WHERE Calendar.Date BETWEEN '2022-01-01' AND '2022-12-31'
		AND Cities.Name = 'Madrid'
	GROUP BY Stores.Name, Products.Name, Calendar.Date
```

Hasta aquí.

Vemos una pauta: estamos uniendo la tabla de hechos con las diferentes tablas de dimensiones que necesitamos. 
Si las miramos en su conjunto, las consultas no parecen excesivamente complejas. Son sólo cadenas a las que vamos añadiendo `JOIN` y filtros.
Repito, si algo tiene bueno el SQL es que sus comandos son exclusivamente cadenas de texto.

Sería posible comenzar con nuestra solución a partir de este tipo de estructuras, pero vamos a intentar dar un paso más allá, vamos
a utilizar [Common Table Expressions](https://en.wikipedia.org/wiki/Hierarchical_and_recursive_queries_in_SQL) (CTE)
para nuestros informes.

Por ejemplo, el último informe (total de ventas por tienda, producto, fecha) lo vamos a traducir por:

```sql
WITH CalendarCte AS
		(SELECT Id, Date
			FROM Dim.Calendar
			WHERE Date BETWEEN '2022-01-01' AND '2022-12-31'
		),
	StoresCte AS
		(SELECT Stores.Id AS StoreId, Stores.Name AS StoreName, Cities.Name AS City
			FROM Dim.Stores INNER JOIN Dim.Cities
				ON Stores.CityId = Cities.Id
					AND Cities.Name = 'Madrid'
		),
	ProductsCte AS
		(SELECT Products.Id AS ProductId, Products.Name AS ProductName
			FROM Dim.Products
		)
	SELECT StoresCte.StoreName, ProductsCte.ProductName, CalendarCte.Date, 
		   SUM(Sales.Quantity) AS Quantity, SUM(Sales.Price) AS Price
		FROM Fact.Sales INNER JOIN CalendarCte
			ON Sales.CalendarId = CalendarCte.Id
		INNER JOIN StoresCte
			ON Sales.StoreId = StoresCte.StoreId
		INNER JOIN ProductsCte
			ON Sales.ProductId = ProductsCte.ProductId
	GROUP BY StoresCte.StoreName, ProductsCte.ProductName, CalendarCte.Date			
```

Como vemos, todas las dimensiones han pasado a ser una CTE. Los filtros y las subentidades han pasado
a formar parte de la misma CTE de dimensión (`StoreCte` engloba `Stores` y `Cities`) y la tabla de 
hechos se relaciona única y exclusivamente con las CTE de dimensiones.

Además, hemos añadido alias a algunas columnas: para no liarnos con el campo `Name` de `Stores` y `Cities`, por ejemplo,
los hemos sustituido por `StoreName` y `City` (en cualquier caso, estamos obligados por el lenguaje, una CTE no nos permite tener dos campos
de salida con el mismo nombre). 

**Nota:** en este caso no es estrictamente necesario porque no utilizamos el nombre de la ciudad, pero si quisiéramos agrupar 
además de por el nombre de tienda, por el nombre de la ciudad, necesitaríamos ese campo, de ahí el ejemplo. Sí, lleváis razón, 
en este caso hemos filtrado por Madrid, pero podríamos querer todas las ciudades, no me seáis tiquismiquis.

Pero ¿en realidad todo esto nos soluciona algo? ¿en qué cabeza cabe que pasar de una consulta de diez líneas a una consulta de veinte líneas
sea una solución mejor?

Hemos hecho el ejemplo más complejo, ahora veamos un ejemplo en el que sólo queremos productos por fecha:

```sql
WITH CalendarCte AS
		(SELECT Id, Date
			FROM Dim.Calendar
			WHERE Date BETWEEN '2022-01-01' AND '2022-12-31'
		),
	ProductsCte AS
		(SELECT Products.Id AS ProductId, Products.Name AS ProductName
			FROM Dim.Products
		)
	SELECT ProductsCte.ProductName, CalendarCte.Date, 
		   SUM(Sales.Quantity) AS Quantity, SUM(Sales.Price) AS Price
		FROM Fact.Sales INNER JOIN CalendarCte
			ON Sales.CalendarId = CalendarCte.Id
		INNER JOIN ProductsCte
			ON Sales.ProductId = ProductsCte.ProductId
	GROUP BY ProductsCte.ProductName, CalendarCte.Date
```

¿Lo veis ahora?

La consulta es *similar* simplemente hemos quitado una CTE (y por supuesto las relaciones).

¿Y si ahora sólo queremos los productos sin filtrar por fecha?

```sql
WITH ProductsCte AS
		(SELECT Products.Id AS ProductId, Products.Name AS ProductName
			FROM Dim.Products
		)
	SELECT ProductsCte.ProductName, SUM(Sales.Quantity) AS Quantity,
		   SUM(Sales.Price) AS Price
		FROM Fact.Sales
		INNER JOIN ProductsCte
			ON Sales.ProductId = ProductsCte.ProductId
	GROUP BY ProductsCte.ProductName
```

Pues quitamos otra CTE y las relaciones / agrupaciones / campos pertinentes.

Y desde aquí, lo único que nos queda es combinar: por tienda, por cliente, por vendedor... todo se reduce a tener más o menos
dimensiones enlazadas con nuestros hechos.

Por supuesto, si sólo quisiéramos el total sin productos y sin fechas, volveríamos a la consulta inicial. Una
consulta sobre la tabla de hechos sin ninguna CTE y sin la cláusula `WITH`.

## Conclusión

Parece que con este análisis es más facil comenzar a trabajar en nuestra solución global: tenemos una forma teóricamente sencilla de generar SQL 
para responder a las consultas que nos habíamos planteado siempre que podamos distinguir entre dimensiones y hechos.

Por supuesto nos queda un largo camino por recorrer. Por ejemplo: parece que la SQL que queremos como resultado no es más que una cadena que podemos 
crear a partir de las definiones de dimensiones y hechos pero ¿de dónde sacamos estas definiciones? 

Lo veremos en el siguiente artículo pero os adelanto: el truco tiene que ver con el esquema y los metadatos de nuestra
base de datos.

## Nota al margen: tablas de calendario

Puede parecer contraproducente tener una tabla adicional sólo para guardar fechas pero la realidad es que
suele ser no sólo habitual si no también muy conveniente.

Quizá lo veamos mejor con un ejemplo. Esta es una tabla de calendario sencilla:

![Tabla de calendario](/blog/articles/development/reporting/images/calendar-table.png)

**Nota:** he obtenido la imagen de [MsSqlTips](https://www.mssqltips.com/sqlservertip/4054/creating-a-date-dimension-or-calendar-table-in-sql-server/) pero
es fácil encontrar otras formas de crear esta tabla si buscáis un poco.

Además de la fecha, tenemos el día de la semana, el nombre del día, el índice del día en el año, el trimestre, la semana ISO... 

Algunos de esos valores los podríamos obtener utilizando funciones de SQL como `DatePart` o similar pero al precalcularlos
y almacenarlos en una tabla evitamos la sobrecarga de la ejecución de ese cálculo al hacer la consulta.

Este calendario es más o menos sencillo pero también podríamos tener en esta tabla campos para relación de las fechas
con otros tipos de calendarios como el [445](https://en.wikipedia.org/wiki/4%E2%80%934%E2%80%935_calendar) o campos que nos indicaran si un 
día es festivo o laborable, etc... que no son sencillos de calcular con funciones de fecha. En algunos casos como la relación con un
calendario 445, directamente no existe esa función.

¿Y para qué queremos estos campos en primer lugar?

Pues para añadirle posibilidades a los informes. Ahora además de filtrar por fechas, también podemos obtener
datos de forma fácil consultando por trimestre o por un día de la semana o por semana ISO sin ningún problema dado que las relaciones
con nuestras tablas de hechos van por el `Id` de la fecha, no por la fecha concreta.

Por supuesto, no sólo filtrar, también podemos agrupar o mostrar estos campos de calendario en nuestros informes sin ningún
cálculo adicional.

Y por si alguien se lo pregunta: sí, también existen tablas de calendarios con fechas y horas o con sólo horas y por el mismo motivo.