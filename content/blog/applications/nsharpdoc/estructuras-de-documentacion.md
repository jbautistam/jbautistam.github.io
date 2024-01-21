+++
title = "Estructuras de documentación"
date = "2019-10-11"
description = "¿Qué son las estructuras de documentación de NSharpDoc?"
thumbnail = "/applications/nsharpdoc/nsharpdoc.jpg"
tags = [ "Aplicaciones" ]
+++

Las **estructuras de documentación** son las estructuras básicas para la generación de documentación
en **NSharpDoc**.

Como hemos explicado en otras secciones de esta Web, [NSharpDoc](/blog/applications/nsharpdoc/nsharpdoc) es un
sistema que nos permite documentar código fuente y otros orígenes de datos frecuentes en el desarrollo de software como
bases de datos o requisitos. Para ello necesitamos una estructura normalizada que hemos llamado precisamente **estructuras
de documentación**.

El esquema de la aplicación de **NSharpDoc** es el siguiente:

![Esquema de la aplicación NSharpDoc](/blog/applications/nsharpdoc/esquema-aplicacion.jpg "Esquema de la aplicación")

En esta imagen vemos en rojo los diferentes 
[proveedores de documentación](/blog/applications/nsharpdoc/proveedores-de-nsharpdoc)
que son las aplicaciones que leen el código fuente o la estructura de base de datos o la fuente XML que se desea documentar
y devuelven **estructuras de documentación** que se pasan al 
[motor de plantillas](/blog/applications/nsharpdoc/plantillas-de-nsharpdoc/plantillas-de-nsharpdoc) que genera los documentos finales.

Por tanto, las **estructuras de documentación** constituyen la salida de los **proveedores de documentación** y la entrada para
los **generadores de documentación**. Esta estructura intermedia nos permite añadir nuevos proveedores y nuevos generadores
sin hacer grandes cambios en el sistema.

Las estructuras de documentación no son más que objetos jerárquicos en formato de árbol que contienen tanto información del
elemento a documentar como parámetros adicionales.

Quizá sea mejor explicarlo con un ejemplo en formato XML:

```XML
<Struct>
	<Name> ProgramStruct </Name>
	<Scope> Global </Scope>
	<Type> Program </Type>
	<Order> 0 </Order>
	<Struct> 
		<Name> Bau.Libraries.LibHelper.API</Name>
		<Scope> Public </Scope>
		<Type> NameSpace </Type>
		<Struct>
			<Name> RegistryApi </Name>
			<Scope> Public </Scope>
			<Type>  Class </Type>
			<Order> 0 </Order> 
			<Parameter Key = "Summary">
				Almacena la información sobre las preferencias de usuario
			</Parameter>
				.....
				
			</Struct>
		</Struct>
	</Struct>
</Struct>
```

En esta estructura, la base es un programa llamado **ProgramStruct**, con un espacio de nombres interno
llamado **Bau.Libraries.LibHelper.API** con una clase llamada **RegistryApy** que contiene el texto de documentación
'Almacena la información sobre las preferencias de usuario'.

El tipo de estructura se almacena en la etiqueta **Type**. En nuestro ejemplo, los diferentes tipos son *Program*, 
*NameSpace*, y *Class*. Este tipo de estructura es la que posteriormente utilizan los elementos `ForEach` de las
[plantillas](/blog/applications/nsharpdoc/plantillas-de-nsharpdoc/plantillas-de-nsharpdoc) para generar la documentación de cada
uno de los elementos de documentación. 	

Por supuesto, este es un ejemplo reducido, en una estructura de documentación real, bajo los elementos de clase tendríamos
métodos o estructuras o enumerados con sus propias estructuras hija como argumentos o propiedades.

Por su parte, si vemos un ejemplo de una estructura de documentación de un esquema de base de datos tenemos lo siguiente:

```XML
<Struct>
	<Name> NorthWind </Name>
	<Scope>Global</Scope>
	<Type> DataBase </Type>
	<Struct>
		<Name> Categories </Name>
		<Scope> Global </Scope>
		<Type> Table </Type>
		<Struct>
			<Name> CategoryID </Name>
			<Scope> Global </Scope>
			<Type> Column </Type>
			<Parameter Key = "Summary"  Reference = "" />
			<Parameter Key = "IsNullable"  Reference = "" > False </Parameter>
			<Parameter Key = "DataType"  Reference = "" > int </Parameter>
			....
		</Struct>
	</Struct>
</Struct>
```

En este esquema dentro de una estructura de tipo **DataBase** llamada **NorthWind** tenemos una estructura de tipo **Table**
llamada **Categories** que contiene una estructura de tipo **Column** llamada **CategoryID** con una serie de parámetros.

Como vemos, **NSharpDoc** no impone una serie de tipos dentro de las estructuras de documentación si no que el programador
del [proveedor](/blog/applications/nsharpdoc/proveedores-de-nsharpdoc) define sus propios tipos y el diseñador
de [plantillas](/blog/applications/nsharpdoc/plantillas-de-nsharpdoc/plantillas-de-nsharpdoc) utiliza estos tipos para definir las salidas.

Este juego de estructuras / plantillas es lo que convierte a [NSharpDoc](/blog/applications/nsharpdoc/nsharpdoc) en un generador 
de documentación válido para diferentes fuentes.
