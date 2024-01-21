+++
title = "BauGridBlazor"
date = "2019-10-11"
description = "Ejemplo de código de grid para Blazor"
thumbnail = "/articles/source-code/baugridblazor/baugridblazor.jpg"
tags = [ "Programación" ]
+++

Para mí, una de las mejores formas de aprender a utilizar una nueva tecnología es desarrollar componentes sobre
ella. Cuando comencé a utilizar Blazor, uno de los componentes que me faltaban era precisamente un grid 'decente',
no sólo una tabla de datos.
	
Hay muchos ejemplos de cómo hacer grids en Blazor y muchos componentes en Github que podemos utilizar como
[GridBlazor](https://github.com/gustavnavar/Grid.Blazor) (realmente de lo mejor
que he visto) o [Table.Net](https://github.com/pablofrommars/TableNet) (en el que
basé mi componente).
	
El primero de ellos, nos ofrece edición, grids anidados, formatos, agrupaciones... pero para mi gusto es excesivamente
complejo. Por ejemplo, añadir un botón a una columna nos obliga a montar un circo interesante. Eso sí, os lo recomiendo
si queréis algo casi profesional.
	
El segundo de ellos es bastante más sencillo, por eso lo elegí como fuente, pero le faltan algunas cosas como la paginación
o la ordenación por múltiples columnas. Por otra parte, aunque ofrece ordenación y filtrado lo hace sobre los elementos cargados, 
es decir, tenemos que tener todos nuestros datos en el cliente antes de ordenar / filtrar. Por supuesto, no es lo que necesito.
	
En ese momento y con ese ejemplo en mente comencé a desarrollar un componente que debía:

* Definir un grid de la forma más sencilla posible.
* Manejar la paginación en servidor.
* Ordenar datos (por varias columnas a la vez).
* Filtrar datos (aunque esto irá a la siguiente iteración).

## Definición de componentes

Una de las mejores cualidades de Blazor es que la generación de componentes es muy sencilla. La idea es tener un
componente que nos defina una tabla (clase `Gridtable`) con su paginación y sus datos.
	
Este componente va a tener como componentes hijo tanto columnas (clase `Column`) que nos van a definir los
elementos de cabecera, como filas, como celdas (clase `Cell`).
	
No me he equivocado, no existe una clase `Row` como tal, es simplemente un parámetro más dentro de la tabla.

De este modo, para colocar un grid en nuestra página, podemos utilizar el siguiente código:

```html
<GridTable Items="_forecasts" Context="wf" Page="@_page" PageSize="@_pageSize" TotalRecords="@_totalRecords" OnChangePage="ChangePageAsync">
	<Loading>
		<p>Loading</p> 
	</Loading>
	<Header>
		<Column Field=@nameof(WeatherForecast.Date) Sortable=true DefaultSort=Column.SortMode.Ascending />
		<Column Field=@nameof(WeatherForecast.TemperatureC) Label="Temp. (C)" Sortable=true />
		<Column Field=@nameof(WeatherForecast.TemperatureF) Label="Temp. (F)" Sortable=true />
		<Column Field=@nameof(WeatherForecast.Summary) Filterable=true />
		<Column Field=@nameof(WeatherForecast.IsReal) Label="Real" />
	</Header>
	<Row>
		<Cell>@wf.Date.ToShortDateString()</Cell>
		<Cell Align=Column.Align.Right>@wf.TemperatureC</Cell>
		<Cell Align=Column.Align.Right>@wf.TemperatureF</Cell>
		<Cell Align=Column.Align.Center>@wf.Summary</Cell>
		<Cell Align=Column.Align.Center>
			<input type="checkbox" disabled readonly checked="@wf.IsReal"/> 
		</Cell>
	</Row>
</GridTable>
```

En este caso, la variable `_forecasts` contiene los datos que vamos a mostrar en el grid. En `_page`, `_pageSize` y `_totalRecords`
se encuentran respectivamente el número de página actual, el tamaó de página y el número total de registros.
	
El evento `OnChangePage` se lanza cada vez que el usuario selecciona una nueva página o cambia la ordenación.

El contexto (`Context = "wf"`) es una estructura de Blazor: al mostrar la colección de elementos, el contexto es cómo se va a llamar la variable
que recoge cada uno de los elementos de la colección (en mi caso `wf`).
	
Por supuesto, en la celda podemos poner el contenido que queramos, incluyendo botones u otros componentes. Eso nos facilita la creación de cuadros
de diálogo para la edición de los elementos.
	
La rutina que maneja el evento `OnChangePage` (en el ejemplo `ChangePageAsync`) es la encargada de recoger los datos que se deben mostrar
en cada página. Lo normal es que lo coja de una API, un archivo o directamente de una base de datos. 
	
Para el ejemplo y por sencillez, me he creado una colección con una gran cantidad de registros en la colección `_forecastsTotal` y cada vez que
se cambia la página u ordenación, simplemente trato esa colección y vuelco la parte adecuada en la colección `_forecasts` para mostrar los datos.
	
El siguiente código es parte del tratamiento del evento de cambio de página:

```csharp
	private async Task ChangePageAsync(BauGrid.EventArguments.GridTablePageEventArgs eventArgs)
	{
		_forecasts = null;
		_page = eventArgs.NewPage;
		_forecasts = SortForecasts(_forecastsTotal, eventArgs.SortFields).Skip((_page - 1) * _pageSize).Take(_pageSize);
	}
```

## Conclusión

Generar componentes en Blazor es muy sencillo, en un día (inspirándonos en código de otros, por supuesto) puedes desarrollar un componente
complejo sin excesivos problemas.

Si quieres ver el código fuente como siempre lo puedes consultar en [GitHub](https://github.com/jbautistam/BauGridBlazor).