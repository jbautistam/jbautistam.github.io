+++
title = "Documentación de base de datos con NSharpDoc"
date = "2019-10-11"
description = "Documentación de bases de datos utilizando NSharpDoc"
thumbnail = "/applications/nsharpdoc/nsharpdoc.jpg"
tags = [ "Aplicaciones" ]
+++

Estas últimas semanas he realizado algunos cambios en el proyecto [NSharpDoc](/blog/applications/nsharpdoc/nsharpdoc),
concretamente en la documentación de los procedimientos de base de datos.

Hasta este momento, el documentador obtenía la estructura de tablas y documentaba sus nombres y descripciones y la relaciones
entre ellos.

Para los procedimientos almacenados y funciones, simplemente escribía las instrucciones, es decir, el contenido del procedimiento
o la función pero no interpretaba los comentarios.

Le he añadido la posibilidad de incluir un resumen de los comentarios iniciales del procedimiento en formato **Markdown**
de forma similar a cómo se documentan los comentarios de documentación de C#.

En este caso, para documentar el procedimiento, simplemente debemos añadirle una cabecera entre caracteres de comentarios (comenzando
por `/**` y terminando por `*/`, observad el doble asterisco inicial):

Por ejemplo:

```SQL
/**
	Este **procedimiento** muestra el tiempo que ha tardado en ejecutar una consulta o un script
	Más información en [la documentación oficial](http://webdocumentacion/logtime)
	Parámetros:
	1. **Step:** nombre del paso
	2. **Description:** descripción del paso
	3. **Start:** hora de inicio de ejecución del paso
	El código de llamada debería ser por ejemplo: 
	
		EXECUTE LogTime 'Primer paso', 'Ejecución completa del primer paso', GetDate()
*/
CREATE PROCEDURE [dbo].[LogTime]
		@Step varChar(2000),
		@Description varChar(2000),
		@Start dateTime
AS
BEGIN
	DECLARE @Records int
	DECLARE @ElapsedMilliseconds int
		-- Obtiene el número de registros de la última consulta y la diferencia en milisegundos
		SET @Records = @@RowCount
		SET @ElapsedMilliseconds = DateDiff(ms, @Start, GetDate())
		-- Desactiva el conteo de registros
		SET NOCOUNT ON
		-- Inserta el log
		INSERT INTO Log (Step, [Description], [Start], [End], ElapsedMilliseconds, Records)
			VALUES (@Step, @Description, @Start, GetDate(), @ElapsedMilliseconds, @Records)
		-- Log
		PRINT @Step
		PRINT @Description + ': ' + 
					CONVERT(varchar, (@ElapsedMilliseconds / 1000) / 60) + ':' + 
					CONVERT(varchar, (@ElapsedMilliseconds / 1000) % 60)
		PRINT '------------------------------------------------------------------------------'
END
```

Genera una documentación de este tipo:

![Documentacion base datos](/blog/applications/nsharpdoc/documentacion-base-datos/documentacion-base-datos.jpg "Documentacion base datos")

Para obtener este resultado, por supuesto hay que cambiar la 
[plantilla de generación](/blog/applications/nsharpdoc/plantillas-de-nsharpdoc/plantillas-de-nsharpdoc), para indicar que un valor se
tiene que interpretar como Markdown añadiéndolo como máscara de la variable. Algo así:
	
```XML
{{Remarks|Markdown}}
```
	
De esta forma, la plantilla de generación para los procedimientos almacenados queda de la siguiente forma:

```XML
<?xml version="1.0" encoding="utf-8"?>
<Page>
	<div class="panel panel-primary">
		<div class="panel-heading">
			<h1 class="panel-title">{{StructType}} {{Name}}</h1>
		</div>
		<IfValue ValueType="Summary">
			<div class="bs-callout bs-callout-info">
				<p>{{Summary}}</p>
			</div>
		</IfValue>
		<IfValue ValueType="Remarks">
			<div class="panel-body">
				{{Remarks|Markdown}}
			</div>
		</IfValue>
	</div>

	<IfValue ValueType="Prototype">
		<div class="panel panel-success">
			<div class="panel-heading">
				<h3 class="panel-title">Definición</h3>
			</div>
			<div class="panel-body">
				<pre class="prettyprint linenums lang-sql">
					<code class="language-sql">
						{{Prototype}}
					</code>
				</pre>
			</div>
		</div>
	</IfValue>
</Page>
```

Aprovechando que estaba modificando partes del documentador, también he cambiado los proyectos de librería para utilizar
.Net Standard y modernizar un poco el proyecto.
	
Como siempre, el código fuente está en [GitHub](https://github.com/jbautistam/NSharpDoc).
