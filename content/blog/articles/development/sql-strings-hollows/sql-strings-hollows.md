+++
title = "Buscando intervalos consecutivos en SQL"
date = "2020-11-22"
description = "SQL para buscar intervalos en una tabla de fechas"
thumbnail = ""
tags = [ "Programación", "SQL" ]
+++

Por mucho que SQL sea un gran lenguaje de consulta de datos hay que reconocer que existen ciertos problemas 
realmente complicados de resolver.


Uno de estos problemas me surge de vez en cuando: buscar intervalos consecutivos de fechas en una tabla.

Imaginad una tabla de ventas básica: productos, fechas y cantidad vendida. 

![Tabla de ventas](/blog/articles/development/sql-strings-hollows/images/SalesIntervalTable.jpg "Tabla de ventas")

La pregunta a las que nos enfrentamos es ¿cuál es el producto que se ha vendido un número mayor de días consecutivos?

En primer lugar, veamos nuestros datos con más detalle, para un producto, partimos de una tabla cómo ésta (las cantidades en realidad no nos interesan, sólo necesitamos
las fechas):

![Fechas por producto](/blog/articles/development/sql-strings-hollows/images/SalesTable.jpg "Fechas de venta para un producto")

Lo primero que queremos saber, son las fechas consecutivas, es decir, nos interesa una tabla más de éste estilo:

![Tabla de intervalos](/blog/articles/development/sql-strings-hollows/images/SalesIntervalTable.jpg "Intervalos de fechas con venta")

Pero, ¿cómo lo logramos a partir de la tabla inicial?

Antes de empezar, permitidme una pequeña licencia: voy a reducir un poco la tabla anterior y me voy a quedar únicamente con el número de día de octubre.
Esto no es realmente necesario, luego lo veremos con fechas y nos daremos cuenta que no hay ninguna diferencia, pero creo que para la explicación
es más fácil verlo así:

![Días con venta](/blog/articles/development/sql-strings-hollows/images/DaySalesTable.jpg "Días de octubre con ventas")

Lo primero que vamos a hacer es numerar las filas:

```sql
SELECT Row_Number() OVER (PARTITION BY ProductId ORDER BY Day ASC) AS RowNumber, ProductId, Day
	FROM Days
```

El resultado de esta tabla es el siguiente:

![Numeración de filas](/blog/articles/development/sql-strings-hollows/images/SqlRowNumber.jpg "Numerando las filas")

**Row_Number** es una función de ventana. Nos va a dar es el número de fila consecutivo particionando por producto, es decir,
si en lugar de un único producto, nuestra tabla tuviera más de un producto (como es de suponer), nos comenzará a numerar de nuevo las filas desde uno
cada vez que cambie de producto.

Con esto no hemos solucionado nada, pero observemos los datos: si restamos el día del número de fila, en caso que sean consecutivos,
nos va a dar el mismo valor. ¿No me creéis?

```sql
SELECT RowNumber, ProductId, Day, Day - RowNumber AS GroupNumber
	FROM (SELECT Row_Number() OVER (PARTITION BY ProductId ORDER BY Day ASC) AS RowNumber,
				ProductId, Day
			FROM Days) AS tmp
```

![Agrupación](/blog/articles/development/sql-strings-hollows/images/SqlGroupNumber.jpg "Agrupando días consecutivos")

Fijáos en la columna **GroupNumber**: es el mismo valor siempre que los datos sean consecutivos. Los días 18, 19 y 20 están en el grupo 17, el día 23 (que no es
consecutivo con los anteriores) está en el grupo 19 y los días 25 a 31 están en el grupo 20.

Es decir, son días consecutivos aquellos que están en el mismo grupo, por tanto ya podemos agrupar para obtener la fecha mínima y máxima de ese intervalo
(la consulta, por supuesto, se puede simplificar pero así creo que es más sencilla a la hora de explicarlo):

```sql
SELECT ProductId, MIN(Day) AS StartDay, MAX(Day) AS EndDay
	FROM (SELECT RowNumber, ProductId, Day, Day - RowNumber AS GroupNumber
			FROM (SELECT Row_Number() OVER (PARTITION BY ProductId ORDER BY Day ASC) AS RowNumber,
						ProductId, Day
					FROM Days) AS tmp
		) AS tmpGroup
	GROUP BY ProductId, GroupNumber
```

Que nos da el resultado buscado:

![Intervalos](/blog/articles/development/sql-strings-hollows/images/SqlInterval.jpg "Intervalos")

Sencillo y elegante ¿verdad? Como decía al principio, la solución no es mía, es un problema relativamente común en las consultas SQL. Se
suelen llamar problemas de cadenas y huecos porque resuelven esas consultas en las que buscamos hechos consecutivos o huecos, es decir, o bien intervalos
de cosas que ocurren de forma consecutiva o bien intervalos de cosas que no ocurren.

En nuestro ejemplo hemos buscando una cadena, fechas consecutivas con ventas, pero también podríamos haber buscado huecos, es decir, días consecutivos
sin ventas que aunque lo parezca, no es trivial.

Para terminar, veamos cómo hacerlo con fechas en lugar de enteros. Todo se reduce a un poco de aritmética con fechas para obtener los grupos,
lo demás es exactamente igual:

```sql
SELECT ProductId, MIN(Date) AS StartDate, MAX(Date) AS EndDate
	FROM (SELECT RowNumber, ProductId, Date, Date_Diff('day', GetDate(), Date) - RowNumber AS GroupNumber
			FROM (SELECT Row_Number() OVER (PARTITION BY ProductId ORDER BY Date ASC) AS RowNumber,
						ProductId, Date
					FROM Days) AS tmp
		) AS tmpGroup
	GROUP BY ProductId, GroupNumber
```

¿Ahora comprendéis por qué me gusta el SQL?

Por cierto, no he respondido a la pregunta, os la dejo a vosotros: ¿cuál es el producto que se ha vendido durante un número mayor de días consecutivos?