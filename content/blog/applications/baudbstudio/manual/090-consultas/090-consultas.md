+++
title = "Consultas"
date = "2019-10-11"
description = "Visualización de resultados de consultas SQL"
thumbnail = "/applications/baudbstudio/manual/090-consultas/090-consultas.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

Mientras escribimos [scripts de SQL](/blog/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql) o
investigamos el contenido de una [conexión](/blog/applications/baudbstudio/manual/020-conexiones/020-conexiones)
a base de datos, necesitaremos sabe cuál es el contenido de nuestras tablas.
	
Para ello, podemos escribir **consultas SQL** y visualizar el resultado en la ventana de consultas.

Para abrir una ventana de consultas, podemos utilizar tanto la opción de menú **Archivo | Nuevo | Consulta** o
el botón **Consulta** de la barra de herramientas, esto nos abrirá una ventana de consulta
como la que vemos al inicio de este artículo.
	
También podemos pulsar dos veces sobre una tabla en la lista de conexiones para abrir una consulta sobre
esa tabla:

![Consulta sobre una tabla](/blog/applications/baudbstudio/manual/090-consultas/consultatabla.jpg "Consulta sobre una tabla")
		   
En la parte superior de esta ventana se encuentra el cuadro de edición donde escribiremos la consulta, la barra
de herramientas colocada en la parte inferior nos permite seleccionar la conexión sobre la que ejecutamos
la consulta y el botón de ejecutar:

![Barra de herramientas de consulta](/blog/applications/baudbstudio/manual/090-consultas/barraherramientas.jpg "Barra de herramientas de consulta")
	
**Nota:** Observe que hay dos botones de ejecución en la aplicación, uno en la ventana de consulta y
otro en la barra de herramientas principal. El botón de la barra de herramientas principal nos permite
ejecutar [scripts de SQL](/blog/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql) y 
[scripts de ETL](/blog/applications/baudbstudio/manual/120-scripts-etl/120-scripts-etl), pero no las consultas de esta ventana. Para ejecutar
consultas debemos utilizar su propia barra de herramientas. La razón principal es que la ejecución de scripts
SQL o ETL puede tardar mucho tiempo y mientras tanto podremos seguir ejecutando en la ventana de consultas sin
necesidad de detener otros procesos.
	
Cuando pulsamos el botón de ejecutar, veremos el resultado de la consulta en el grid inferior:

![Visualización de resultados](/blog/applications/baudbstudio/manual/090-consultas/resultadosconsulta.jpg "Visualización de resultados consulta")
			
Junto al botón de ejecutar, vemos un botón etiquetado como **Plan de ejecución**, cuando lo pulsemos, veremos en el cuadro de texto
de la ficha **Plan de ejecución** el plan que va a ejecutar el servidor de base de datos para obtener el resultado de la consulta:
	
![Plan de ejecución](/blog/applications/baudbstudio/manual/090-consultas/planejecucion.jpg "Plan de ejecución de una consulta")

La última ficha nos permite visualizar un gráfico básico de la consulta (esta opción aún esta en desarrollo, por ahora es algo limitada):

![Gráfico](/blog/applications/baudbstudio/manual/090-consultas/graficoconsulta.jpg "Gráfico de una consulta")

Además, el botón de **Guardar** nos permite guardar el resultado de la consulta en un archivo 
[CSV o parquet](/blog/applications/baudbstudio/manual/095-archivos-de-datos/095-archivos-de-datos) mientras que el botón de
**Paginar consulta** lanzará automáticamente consultas paginadas sobre el servidor.
	
Las consultas que escribamos en esta ventana, utilizan el mismo 
[archivo de parámetros](/blog/applications/baudbstudio/manual/045-archivos-de-parametros/045-archivos-de-parametros) que los
scripts SQL, es decir, podemos utilizar los valores de los parámetros y constantes de los archivos de parámetros 
en esta ventana.
	
Por ejemplo, podemos escribir esta consulta sobre una conexión a SqLite:

```SQL
SELECT [Orders].[OrderID], [Orders].[CustomerID], [Orders].[EmployeeID], [Orders].[OrderDate], 
		[Orders].[RequiredDate], [Orders].[ShippedDate], [Orders].[ShipVia], 
		[Orders].[Freight], [Orders].[ShipName], [Orders].[ShipAddress], [Orders].[ShipCity], 
		[Orders].[ShipRegion], [Orders].[ShipPostalCode], [Orders].[ShipCountry]
	FROM [Orders]
	WHERE OrderDate > @StartDate
```

Donde **StartDate** está definida en nuestro archivo de parámetros como:

```JSON
{
	"StartDate": "1970-01-01"
}
```
	
El resultado que obtenemos en este caso será:

![Resultado de una consulta](/blog/applications/baudbstudio/manual/090-consultas/resultadosconsultaparametros.jpg "Resultado de una consulta con parámetros sobre SqLite")

Por último, para facilitar la escritura de consultas:


* Si arrastramos y soltamos una tabla desde las conexiones al editor de consultas se escribirá el nombre de la tabla.
* Si arrastramos y soltamos una tabla desde las conexiones al editor de consultas mientras mantenemos pulsada
la tecla de mayúsculas, se escribirá en la consulta el nombre de la tabla y todos sus campos.
		
Del mismo modo que en la ventana de [edición de scripts](/blog/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql),
si seleccionamos un bloque de código, se ejecutará una consulta sólo sobre el bloque seleccionado.