+++
title = "Generación de informes"
date = "2024-01-20"
description = "Generación de informes: parte 1"
thumbnail = "/images/noimage.jpg"
tags = [ "Development", "Reports" ]
+++

Posiblemente y para mi desgracia, lo que más he desarrollado durante mi vida profesional han sido sistemas de generación de informes. El segundo lugar debe 
corresponderle al mantenimiento de usuarios pero esa es otra historia, centrémonos en los informes.

Hay muy pocas aplicaciones de gestión para las que antes o después no nos pidan obtener consultas de la base de datos, exportar los datos a archivos 
o similar.

**Un aviso:** este artículo no tiene código funcional, es sólo una introducción. Os explicaré el problema que quiero solucionar 
al final y dejaremos el código para los siguientes artículos.

Mi forma de gestionar los informes desde hace varios proyectos es bastante similar y tampoco tiene ningún misterio: SQL e `IDataReader`.

Podría resumirlo como: *intentemos no complicarnos la vida*.

La idea es que desde negocio, antes o después, nos van a solicitar informes, algunos de ellos serán simples y otros muy
complicados, los necesitan rápido, para consumirlos en el momento y de vez en cuando, hasta les gustaría
crearse sus propios informes.

Con esos requisitos, suelo crear una tabla de `Reports` en mi base de datos, con esta estructura:

| Reports |
|---------|
| Id      |
| Name    |
| SQL     |

Sencillo ¿verdad? Así sólo tenemos que leer la tabla y mostrar los diferentes informes en un formulario y que el usuario
seleccione el listado que desea ver. Una vez que el usuario ha seleccionado el listado, ejecuto la consulta SQL y obtengo un `IDataReader`:

```csharp
await connection.ExecuteReaderAsync(sql, null, System.Data.CommandType.Text, TimeSpan.FromMinutes(5), cancellationToken);
```

este `IDataReader` lo puedo transformar según me convenga, por ejemplo, puedo generar un Json con una estructura similar a esta:

```json
{
	"headers": [ "header1", "header2" ... ],
	"rows":
		[
			[ "row 1 - value1", "row 1 - value2" ...],
			[ "row 2 - value1", "row2 - value2", ... ]
		]
}
```

y representarla en el front utilizando Blazor por ejemplo.

**Nota:** normalmente es más complicado, pero ya nos entendemos.

Si por ejemplo en lugar de mostrarlo en el front, quiero exportarlo a CSV puedo hacer lo siguiente (escrito de memoria sin compilar ni nada parecido):

```csharp
/// <summary>
///		Escribe un <see cref="IDataReader"/> en un archivo Csv
/// </summary>
public void Write(string fileName, IDataReader reader)
{
	StringBuilder builder = new();

		// Añade la cabecera de la tabla de datos
		CreateHeaderRow(builder, reader);
		// Añade las filas
		CreateDataRows(builder, reader);
		// Escribe los datos
		File.WriteAllText(fileName, builder.ToString());
}

/// <summary>
///		Crea una fila de cabecera para el CSV con las columnas del <see cref="IDataReader"/>
/// </summary>
private void CreateHeaderRow(StringBuilder builder, IDataReader reader)
{
	// Añade los nombres de los campos
	for (int index = 0; index < reader.FieldCount; index++)
	{
		builder.Append(reader.GetName(index));
		if (index != reader.FieldCount - 1)
			builder.Append(",");
	}
	// Añade el salto de línea
	builder.AppendLine();
}

/// <summary>
///		Crea los datos para las filas del <see cref="IDataReader"/>
/// </summary>
private void CreateDataRows(StringBuilder builder, IDataReader reader)
{
	while (reader.Read())
	{
		// Añade los valores del registro
		for (int column = 0; column < reader.FieldCount; column++)
		{
			builder.Append(reader[column]?.ToString());
			if (column != reader.FieldCount - 1)
				builder.Append(",");
		}
		// Añade el salto de línea
		builder.AppendLine();
	}
}
```

Por supuesto no lo hagáis así: para exportar a CSV suelo utilizar la librería de la que hablábamos
en el artículo de [optimización de lectura de CSV](/blog/articles/development/optimising-csv-reader/optimising-csv-reader) 
que podéis encontrar en [GitHub](https://github.com/jbautistam/LibCsvFiles) que recibe un `IDataReader` y escribe
un archivo. Si quiero exportar a archivos Parquet, suelo utilizar [mi librería de parquet](https://github.com/jbautistam/LibParquetFiles) 
basada en [Parquet DotNet](https://github.com/aloneguid/parquet-dotnet) que 
también recibe un `IDataReader` y para exportar a Excel, suelo utilizar [otra de mis librerías](https://github.com/jbautistam/LibExcelFiles) 
con el mismo funcionamiento.

**Nota:** hay muchas soluciones similares, escoged la que más os guste.

Pero bueno, no todo puede ser tan sencillo: si queremos un generador de informes más o menos funcional vamos a necesitar que el usuario 
seleccione los datos que desee. No nos sirve de nada un informe que nos muestre en pantalla los 800.000 registros de transacciones
de cliente que tenemos en el sistema. Debemos permitir al usuario seleccionar y filtrar lo que realmente quiere ver.

Por tanto, necesitamos que cada informe tenga su lista de parámetros. Para eso nos creamos una tabla adicional, `ReportParameters`
con los parámetros que el usuario puede rellenar con una estructura similar a esta:

| ReportParameters |
|------------------|
| Id               |
| ReportId         |
| Type             |
| Name             |
| Required         |
| Default          |
| SqlVariable      |

En cada informe podemos tener varios parámetros con su tipo (enteros, cadenas, valores lógicos...), su nombre, el
valor predeterminado, etc... Con estos datos ya podemos mostrar en la pantalla los controles específicos para cada parámetro y cuando
el usuario introduzca los datos, podemos filtrarlos.

Con este nuevo requisito, tenemos que modificar la cadena SQL del registro de la tabla `Reports` para
que considere los parámetros de esa tabla. Por ejemplo, para un informe de clientes por delegación:

```sql
SELECT Customers.Name, Customers.LastName, Customers.Address, Delegations.Name
	FROM Customers INNER JOIN Delegations
		ON Customers.DelegationId = Delegations.DelegationId
	WHERE (@CustomerName IS NULL
				OR Customers.Name LIKE '%' + @CustomerName + '%')
		AND (@DelegationName IS NULL
				OR Delegations.Name LIKE '%' + @DelegationName + '%')
```

Tenemos dos parámetros `@CustomerName` y `@DelegationName` que se corresponderán con dos registros de la tabla de parámetros del informe:

| Id | ReportId | Type   | Name       | Required | Default  | SqlVariable      |
|----|----------|--------|------------|----------|----------|------------------|
| 1  |  2       | string | Customer   | false    |          | @CustomerName    |
| 2  |  2       | string | Delegation | false    |          | @DelegationName  |

Para obtener el informe sólo tenemos que añadir los valores de estos parámetros que hayamos recogido del usuario y pasárselo
a nuestra SQL.

Según pase el tiempo, nuestra generación de informes se complicará: podemos añadir una tabla de columnas
para variar las ordenaciones de salida, podemos modificar la SQL para que permita paginar con `LIMIT / OFFSET`, podemos añadir
tablas para asignar formatos a las columnas (no sólo colores si no también formatos numéricos, por ejemplo), podemos añadir
tipos de SQL asociadas a los parámetros para que el usuario pueda seleccionar en un combo la delegación en lugar
de introducir un nombre de delegación, podemos indicar diferentes conexiones a bases
de datos, podemos añadir seguridad por fila añadiendo a la SQL información del rol del usuario... 
lo que se nos ocurra.

Lo importante es que tenemos una abstracción sencilla que puede evolucionar y nuestro mantenimiento
se ha reducido a añadir registros a una o varias tablas (o a varios archivos o a una serie de JSON... ya nos entendemos).

## Clases

Pero ¿por qué no tenemos clases separadas por informe? Quizá os parezca una pregunta absurda pero la verdad es que parece lógico,
sobre todo a aquellos que estamos acostumbrados a acceder a la base de datos utilizando ORMs: si tengo un informe de clientes 
¿por qué no tener una clase de clientes para generar el informe? ¿no será mejor para cumplir el principio de mínima responsabilidad? 

Pues depende de cómo lo mires: si a partir de tus requisitos de negocio obtienes tareas como "preparar un informe de clientes", 
"preparar un informe de stock", "preparar un informe de ventas",
"preparar un informe de ganancias de la empresa en 2.023"... puede que tenga lógica pensar que cada uno de ellos 
se deba implementar con clases diferentes.

Lo malo es que a partir de esa división en tareas, te vas a enfrentar a cuatro problemas diferentes, uno por informe y según evolucione el proyecto, 
acabarás encontrándote con tantos problemas como informes. Que todos los problemas
sean conceptualmente iguales, no significa que dejes de tener infinitos problemas. Que puedas copiar, pegar y modificar código
no lo hace precisamente mejor, simplemente lo hace más largo.

¿Tu "mínima responsabilidad" no debería plantearse de otra forma? ¿No debería ser "generar informes" en lugar de
"generar el informe X"?

Si te quedas con la idea que necesitas un generador de informes, todo es más sencillo: sólo tienes SQL que no es más 
que texto que puedes parametrizar con el cual obtienes un `IDataReader` que puedes transformar en:

* Una tabla de datos para un control `ListView` en WPF o Windows Forms (o como se llame en tu sistema de interface de usuario).
* Una cadena de JSON para enviárselo de vuelta a un navegador o a tu frontend.
* Un archivo Excel.
* Un archivo Parquet.
* Un archivo CSV.

¿Te has parado en pensar lo que significa hacer todo eso con clases independientes?

* Cargar `IEnumerable<Client>` en `ListView`.
* Cargar `IEnumerable<Sales>` en `ListView`.
* Convertir `IEnumerable<Client>` a Json.
* Convertir `IEnumerable<Sales>` a Json.
* Exportar `IEnumerable<Client>` a Excel.
* Exportar `IEnumerable<Sales>` a Excel.
* Exportar `IEnumerable<Client>` a CSV.
* Exportar `IEnumerable<Sales>` a CSV...

Te obliga a programar, cada informe necesitará su propio diseño y su propia implementación. Si ahora quieres
cambiar el formato de salida del Excel de clientes, prepárate a cambiar, recompilar, probar y subir de nuevo el código.

Por supuesto: escribir SQL también es programación, pero una vez desarrollada la consulta,
si sigues los pasos anteriores, no tienes que cambiar tu código. Todo es una cadena de texto que puedes almacenar en 
la base de datos o en un archivo externo y que puedes modificar, añadir o eliminar sin necesidad de recompilar
tu aplicación. Todo SQL te devuelve un `IDataReader` que puedes transformar, exportar, mostrar.

Simple. No le busques más vueltas.

## ¿Esto es todo?

No, en realidad, aquí empieza todo. Como hemos dicho, el generador de informes sigue creciendo y comenzamos a ver un problema.

El departamento de negocio ahora quiere estos informes:

1. Las ventas de los vendedores entre dos fechas.
2. Las compras de los clientes entre dos fechas.
3. Las ventas en una tienda entre dos fechas.
4. Las ventas de los vendedores por cada tienda entre dos fechas.
5. Las compras de los clientes por cada tienda entre dos fechas.
6. Las compras de los clientes por cada vendedor en cada tienda entre dos fechas.
7. Las compras de los clientes por cada vendedor en cada tienda de Madrid entre dos fechas...

¿Véis el problema? Nos piden n informes, pero en realidad no es más que un informe en el cual el usuario desea
seleccionar diferentes entidades para los mismos datos.

Tenemos cuatro entidades: vendedores, clientes, tiendas y ciudades y un dato: la cantidad vendida y queremos obtener diferente
información con arreglo a esas entidades / datos.

**Nota:** existe otra entidad, la fecha, pero nos entendemos.

El caso es que dependiendo de las funcionalidades de nuestro proyecto, los informes siempre van a girar sobre varias
entidades principales y varios datos de negocio.

¿No hay una forma más sencilla de generarlos? Precisamente para eso existen la herramienta de BI: una herramienta
que nos permite generar informes a partir de nuestros datos sin necesidad de escribir SQL (o esa es la idea, el SQL o similar está
por ahí escondido en alguna parte).

¿Podría conseguir algo así?

La respuesta, por supuesto es sí pero posiblemente no con la estructura de tablas / documentos que tenemos ahora, vamos a necesitar
muchas más cosas. Ese será el contenido de los siguientes artículos.

Como decía al principio la única razón de éste post era sembrar la curiosidad, en el resto de artículos veremos una forma
de crear una librería que nos permita generar informes más complejos y versátiles (o eso espero).
