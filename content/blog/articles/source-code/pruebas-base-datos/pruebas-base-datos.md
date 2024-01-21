+++
title = "A vueltas con las pruebas de base datos"
date = "2019-10-11"
description = "Motor para gestión de pruebas sobre base de datos"
thumbnail = "/articles/source-code/pruebas-base-datos/pruebas-base-datos.jpg"
tags = [ "Programación" ]
+++

Dentro del mundo de las pruebas del software, las pruebas de base de datos, para mí, son una de las más complejas.

Aunque existe mucha documentación sobre cómo debemos realizar este tipo de pruebas, conozco muy pocos
entornos actualizados que nos permitan automatizar este tipo de testing.

**Nota:** al contrario de lo que suelo mostrar en este blog, esta vez no se trata de una aplicación
completa si no de una aplicación aún en desarrollo, por eso no publico aún enlaces a GitHub. Este
artículo precisamente es para solicitar ideas o detectar fallos en mi forma de plantear el proceso.
Cuando la aplicación esté completa se volcará a GitHub, como el resto. 

Los pasos para probar una base de datos, son prácticamente los mismos que para cualquier tipo
de aplicación:
	
* Preparación de datos
* Ejecución sobre la base de datos
* Comprobación de resultados
	
La preparación de datos implica crear las estructuras necesarias que imiten nuestra base
de datos final, es decir, crear el mismo esquema que en la base de datos de producción.
	
En esta base de datos de pruebas tenemos que incluir una serie de datos conocidos, normalmente a partir de
scripts o copiando una serie de datos semilla.
	
Sobre esa base de datos podemos ejecutar nuestros scripts o instrucciones de modificación para por último
comparar con los resultados que deseamos.
	
La comparación o comprobación de resultados lo podemos hacer, con consultas sobre la propia base de datos. Por ejemplo,
si hemos emitido una modificación del precio del registro `20` de la tabla `Order details` al valor `232`, 
podríamos comprobar si la modificación es correcta con una consulta de este estilo:

```SQL
SELECT COUNT(*)
	FROM [Order details]
	WHERE OrderId = 20
		AND Price = 232
```

Lógicamente, si la modificación ha sido correcta nos debería devolver un valor mayor que cero.

Otra forma de hacer este tipo de comprobaciones es crear una tabla adicional en el esquema con los resultados esperados
y solicitar una consulta de este estilo:
	
```SQL
SELECT COUNT(*)
	FROM [Result customers] LEFT JOIN Customers 
		ON [Result customers].CustomerId = Customers.CustomerId
	WHERE [Result customers].CompanyName <> COALESCE(Customers.CompanyName, '')
```

Hasta ahora se me ocurre sólo otro tipo de pruebas: comprobar si se puede ejecutar o no una instrucción sobre la base
de datos. Este tipo de pruebas nos serviría para identificar, por ejemplo, si la base de datos detecta que una instrucción
de modificación cumple las restricciones de clave foránea. 
	
Por tanto, un entorno de pruebas de base de datos, para mí debería incluir:

* Métodos para crear una base de datos de prueba o bien para copiar los datos de una base de datos "semilla" 
a la base de datos que utilizarmos para la prueba. De esta forma no ensuciamos la base de datos inicial 
y podemos utilizarla de nuevo para otras pruebas.
* Métodos para ejecución de scripts sobre la base de datos.
* Métodos para comprobar los resultados o los errores de ejecución.

### El entorno de ejecución 

Dado que lo que me gustaría es que nuestro trabajo de exportación de datos / pruebas fuese suficientemente flexible como
para ejecutarlo en diferentes entornos, prácticamente toda la aplicación está enfocada a ser dirigida por
scripts. 
	
Para no liar demasiado la aplicación con un lenguaje propio (y para no liarme con analizadores sintácticos) he decidido
utilizar XML como base para todos los scripts de prueba.
	
Lo primero que necesita una aplicación de este estilo es reconocer las bases de datos. Para ello utilizo un archivo de
conexiones de este tipo:
	
```xml
<?xml version="1.0" encoding="utf-8"?>
<DbScriptsConnections>
	<Connection Key="NW" Type = "SqlServer">
		<Name>SqlServer - Northwind</Name>
		<Description/>
		<ConnectionString>Data Source=localhost;user=sa;password=xxxx;Initial Catalog=Northwind</ConnectionString>
	</Connection>
	<Connection Key="TS" Type = "SqlServer">
		<Name>SqlServer - TestResult</Name>
		<Description/>
		<ConnectionString>Data Source=localhost;user=sa;password=xxxx;Initial Catalog=NorthwindTest</ConnectionString>
	</Connection>
</DbScriptsConnections>
```

En este caso se van a utilizar conexiones a bases de datos de tipo SqlServer (la famosa Northwind). La primera conexión (NW) será
nuestra base de datos semilla, la segunda (TS) será la base de datos sobre la que ejecutaremos las pruebas.

**Nota:** La aplicación por ahora está preparada para trabajar con Sql Server y SqLite aunque
debo reconocer que la mayoría de las pruebas de implementación las he hecho sobre Sql Server. La idea es añadir
más proveedores de base de datos (quién sabe si utilizando [MEF](/blog/articles/source-code/plugins-2/plugins-2), por ejemplo).

Una vez identificadas las conexiones, el segundo paso es definir los scripts y el orden que se va a ejecutar:

```xml
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Name>Test data</Name>
	<Parameter Key="OrderId" Type="numeric" Value="3"/>
	<PathProject>C:\Test\Scripts\Sample</PathProject>

	<!-- Proceso de scripts de base de datos: Exportación de Northwind a NorthwindTs -->
	<Step StartWithPreviousError = "false">
		<Name>Scripts database process: export NW to TS</Name>
		<Script>DbScripts\01_NW_Export_To_TS.xml</Script>
	</Step>

	<!-- Proceso de scripts de pruebas sobre la base de datos: ejecución de comandos -->
	<Step StartWithPreviousError = "false">
		<Name>Test database execute</Name>
		<Script>DbScripts\05_TS_Test_Execute.xml</Script>
	</Step>

	<!-- Proceso de scripts de pruebas sobre la base de datos: lectura de escalares (SELECT COUNT) -->
	<Step StartWithPreviousError = "false">
		<Name>Test database execute scalar</Name>
		<Script>DbScripts\10_TS_Test_SelectScalar.xml</Script>
	</Step>
</DbScript>
```

**Nota:** las conexiones están separadas de los scripts de ejecución porque un script de pruebas
puede servir para diferentes tipos de base de datos. Dado que el motor realmente no es sólo para pruebas 
y se puede utilizar para exportación o transformación de datos parece lógico que las conexiones sean independientes,
así podemos lanzar los mismos pasos de ejecución sobre diferentes bases de datos simplemente añadiendo otro archivo de
conexión.
	
En este archivo de prueba, tenemos tres scripts. El primero de ellos, *01_NW_Export_To_TS.xml** es el paso
de exportación de base de datos, el segundo, #em 05_TS_Test_Execute.xml* son las pruebas para instrucciones
de modificación de base de datos, el último, *10_TS_Test_SelectScalar.xml* son las pruebas para instrucciones
de consulta escalar.
	
Además, en este archivo indicamos cuál es el directorio de proyecto (el directorio base donde se encuentran los scripts) y
parámetros de ejecución globales (en este caso *OrderId*).
	
En este ejemplo no es necesario añadir parámetros puesto que es bastante sencillo, pero lo he incluido porque en algunos
casos los parámetros globales nos permiten que un script se ejecute bajo ciertas condiciones, por ejemplo, que sólo
se transfieran los datos de un pedido en concreto.
	
Aparte de los parámetros globales, el script también admite parámetros locales para cada uno de los pasos. Por ahora
baste saber que existen, en próximos artículos, cuando veamos las instrucciones condicionales o los bucles profundizaremos
en este tipo de argumentos.
	
Echemos un vistazo al script de copia de datos (`01_NW_Export_To_TS.xml`):

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Block Name="Empty tables">
		<Execute Target ="TS">
			DELETE FROM [Order Details];
			DELETE FROM [Orders];
			DELETE FROM [Shippers];
			DELETE FROM [Products];
			DELETE FROM [Suppliers];
			DELETE FROM [Categories];
			DELETE FROM [Employees];
			DELETE FROM [CustomerDemographics];
			DELETE FROM [CustomerCustomerDemo];
			DELETE FROM [Customers];
			DELETE FROM [EmployeeTerritories];
			DELETE FROM [Employees];
			DELETE FROM [Territories];
			DELETE FROM [Region];
		</Execute>
	</Block>

	<Block Name="Copying regions">
		<BulkCopy Source="NW" Target="TS" Table="Region">
			SELECT RegionID, RegionDescription
				FROM Region
		</BulkCopy>
	</Block>

	<Block Name="Copying territories">
		<BulkCopy Source="NW" Target="TS" Table="Territories">
			SELECT TerritoryID, TerritoryDescription, RegionID
				FROM Territories
		</BulkCopy>
	</Block>
	
	....
	
</DbScript>
```

He cortado el script pero el resto de instrucciones es igual para copiar el resto de tablas. Vamos a ver cada una de ellas:

* La instrucción `Block` es opcional y se utiliza para agrupar instrucciones y para log (la consola imprime el mensaje
del bloque y mide el tiempo de ejecución).
* La instrucción `Execute` ejecuta una o varias sentencias sobre la base de datos identificada en el atributo
`Target` (en este caso la base de datos de pruebas). Las instrucciones `Execute` no esperan una colección
de datos. Existen otras instrucciones para consulta pero están fuera del ámbito de esta introducción.
* La instrucción `BulkCopy` copia los datos de una consulta sobre la base de datos identificada en el atributo
`Source` (en este caso NW - Northwind) sobre la base de datos identificada en el atributo `Target`
en la tabla indicada en el atributo `Table`.
		
El script es bastante sencillo, borra los datos que pudieran existir anteriormente en nuestra base de datos de prueba
y copia los datos de la base de datos origen. Utiliza un bulkcopy para hacer la copia. 
	
En mi equipo con SQL Server sobre Docker, la copia de datos tarda unos cinco segundos aunque eso no nos diga mucho
dado que Northwind es una base de datos bastante pequeña. Las pruebas
que he hecho con bases de datos más grandes son razonables, afortunadamente parece que depende más de la instrucción
bulkcopy que de mi maestría como optimizador.

En lugar de utilizar instrucciones de borrado sobre una base de datos con un esquema predefinido, podríamos haber
usado instrucciones para crear por completo la base de datos utilizando instrucciones `CREATE TABLE`
pero para un ejemplo lo veía más claro así.

Observad el orden de las sentencias `DELETE` y de copia de datos: dado que hay referencias entre tablas, el borrado
y la inserción se debe hacer utilizando el orden lógico de la base de datos.
	
**Nota:** posiblemente una de las primeras mejoras debería ser copiar esquema y datos sin necesidad
de ir instrucción por instrucción pero dado que SQL Server nos da una opción maravillosa de "Generar script de base de datos"
no me parecía un punto a tener en cuenta al comienzo del desarrollo.
	
Una vez ejecutado este script, se pasaría al siguiente `05_TS_Test_Execute.xml` que prueba las instrucciones de modificación:

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Block Name="Assert update">
		<AssertExecute Target ="TS" WithError="false" Records="1">
			UPDATE Customers
				SET CompanyName = CompanyName + ' Updated'
				WHERE CustomerId = 'ALFKI'
		</AssertExecute>
	</Block>

	<Block Name="Assert delete no records">
		<AssertExecute Target ="TS" WithError="false" Records="0">
			DELETE FROM Customers
				WHERE CustomerId = 'AQSSXXXXLFKI'
		</AssertExecute>
	</Block>

	<Block Name="Expected error when truncate table">
		<AssertExecute Target ="TS" WithError="true" Records="0">
			TRUNCATE TABLE Customers
		</AssertExecute>
	</Block>

	<Block Name="Error when test TRUNCATE TABLE and we wait a correct result">
		<AssertExecute Target ="TS" WithError="false" Records="0">
			TRUNCATE TABLE Customers
		</AssertExecute>
	</Block>
</DbScript>
```

Las instrucciones `AssertExecute` indican que se debe ejecutar una instrucción sobre la base de datos `Target` e
informar de si el resultado es o no erróneo (atributo `WithError`).
	
Si se ha ejecutado correctamente, se comprueba el número de registros afectados (predefinido en el atributo `Records`).

En ejecución, las instrucciones `AssertXXX` lanzan un evento especial que el ejecutable muestra en la consola indicando
si ha habido o no un error en la prueba.

El último script `10_TS_Test_SelectScalar.xml`, realiza las comprobaciones de consultas escalares:

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Block Name="Assert customers with key ALFKI">
		<AssertScalar Target ="TS" Result="1">
			SELECT COUNT(*)
				FROM Customers
				WHERE CustomerId = 'ALFKI'
		</AssertScalar>
	</Block>

	<Block Name="Assert customers with key XXXXX">
		<AssertScalar Target ="TS" Result="0">
			SELECT COUNT(*)
				FROM Customers
				WHERE CustomerId = 'XXXXX'
		</AssertScalar>
	</Block>

	<Block Name="Assert customers: error">
		<AssertScalar Target ="TS" Result="25">
			SELECT COUNT(*)
				FROM Customers
				WHERE CustomerId = 'XXXXX'
		</AssertScalar>
	</Block>
</DbScript>
```

La instrucción `AssertScalar`, ejecuta una consulta escalar, que devuelve únicamente un número, sobre la base de datos
`Target` y comprueba si el resultado es el mismo que el valor del atributo `Result`.
	
Por supuesto, nada nos impide mezclar en un script instrucciones `AssertExecute` con `AssertScalar`
pero para los ejemplos me resultaba más sencillo separarlos.

### Conclusiones

Aparte de las instrucciones que hemos visto hasta ahora, la librería incluye sentencias para realizar consultas o
transformar datos además de consultas de prueba, esto hace que el motor sea más flexible. Según avance el desarrollo
iré publicando los diferentes tipos de instrucciones.

Como decía al principio, esta aplicación continua en desarrollo. Aún le faltan algunas cosas:

* Integración de otros proveedores de bases de datos.
* Ejecución de scripts SQL: si tenemos un script completo para ejecutar sobre una base de datos y deseamos
hacer pruebas con él, en este momento lo tenemos que pasar a instrucciones XML. Esto nos trae 
complicaciones a la hora de desarrollar puesto que nos obliga a tener un código para ejecución en producción 
sobre la base de datos y otro para pruebas. Sería interesante poder
ejecutar el script completo sin variaciones y posteriormente ejecutar los `Assert` adecuados.
* Falta la documentación del resto de instrucciones: if, while, load... y sobre las diferentes transformaciones de datos.
* La importación / exportación de CSV aún necesita más pruebas.

Seguimos trabajando en ello. Nuevas noticias en breve.