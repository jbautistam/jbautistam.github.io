+++
title = "Proveedores de NSharpDoc"
date = "2019-10-11"
description = "Introducción a los proveedores de documentación de NSharpDoc"
thumbnail = "/applications/nsharpdoc/nsharpdoc.jpg"
tags = [ "Aplicaciones" ]
+++

[NSharpDoc](/blog/applications/nsharpdoc/nsharpdoc)) es un sistema extensible para generación de documentación asociada
al desarrollo, es decir, permite añadir nuevos módulos o métodos para documentar diferentes
sistemas como bases de datos u otros lenguajes de programación distintos a C#.

Para permitir la implementación de otras **fuentes de documentación** en **NSharpDoc**, se utilizan los conceptos
de proveedores que explicamos en este artículo y de 
[estructuras de documentación](/blog/applications/nsharpdoc/estructuras-de-documentacion)
y [plantillas](/blog/applications/nsharpdoc/plantillas-de-nsharpdoc/plantillas-de-nsharpdoc)]
que se detallan en otras secciones de esta Web.

Los proveedores de **NSharpDoc** actualmente no son más que clases que esperan una serie de parámetros y devuelven
una colección de [estructuras de documentación](/blog/applications/nsharpdoc/estructuras-de-documentacion)
a partir de las que **NSharpDoc** puede generar la documentación basándose en **plantillas**.

Los proveedores de **NSharpDoc** deben implementar la interface `IDocumenter` que se define en el espacio de nombres
`Bau.Libraries.LibNSharpDoc.Models.Interfaces`. En esta interface tenemos un único método llamado `Parse` que
espera una colección de parámetros de tipo `StructParameterModelDictionary` y devuelve un objeto 
`StructDocumentationModel` con el resultado de la documentación (es decir, las 
[estructuras de documentación](/blog/applications/nsharpdoc/estructuras-de-documentacion).

La clase `StructParameterModelDictionary` contiene los parámetros para la generación de la documentación 
en un diccionario de tipo *Clave - Valor* que depende del proveedor dado que cada uno de ellos necesita 
argumentos diferentes. Un proveedor para documentación de código fuente sólo necesita el nombre de proyecto 
o solución mientras que un proveedor para documentación de base de datos puede requerir la cadena de conexión 
o el nombre de servidor, usuario y contraseña por ejemplo.

Por el momento, **NSharpDoc** no utiliza ningún modelo de inyección de contenido para importar nuevos procesos de documentación
si no que mantiene las referencias a los diferentes proveedores directamente en el código de la librería `LibNSharpDoc.Processor`.
Esto, por supuesto, limita la capacidad de extender el sistema puesto que obliga a añadir nuevas referencias en el propio proyecto
y recompilar el sistema antes de poder utilizarlas. Aún no estoy seguro de si implementar un modelo de inyección de contenido
o plugins sobre la librería de proceso o bien crear un sistema de paso de información entre diferentes consolas. El primer
sistema nos daría un 'ecosistema' de proveedores pero limita las posibilidades para creación de proveedores a sistemas
escritos dentro del framework.NET mientras que un sistema de consolas o ejecutables nos daría la posiblidad de crear
extensiones en cualquier lenguaje de programación o framework aunque de un forma más 'fea' en sentido arquitectónico.