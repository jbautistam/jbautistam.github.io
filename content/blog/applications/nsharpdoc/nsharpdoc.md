+++
title = "NSharpDoc"
date = "2019-10-11"
description = "NSharpDoc: herramienta para documentación de código fuente C# y bases de datos"
thumbnail = "/applications/nsharpdoc/nsharpdoc.jpg"
tags = [ "Aplicaciones" ]
+++

**NSharpDoc** es una aplicación para documentación de código fuente y bases de datos.

En esta primera versión de **NSharpDoc**, podemos documentar de forma sencilla tanto código fuente en C# como
bases de datos SQL Server.

En el futuro, esperamos añadir otros lenguajes de programación así como otros servidores de bases de datos o
diferentes fuentes de documentación interesantes para el desarrollo de software como por ejemplo las capturas de requisitos.

## Funcionamiento de la aplicación

Al ejecutar **NSharpDoc** aparece la pantalla principal de la aplicación: 

![Pantalla principal de NSharpDoc](/blog/applications/nsharpdoc/pantalla-principal.jpg "Pantalla principal")

**NSharpDoc** nos permite la gestión de **proyectos de documentación** es decir, la unión de varias fuentes de documentación
como bases de datos o código fuente en un único proyecto o sitio de documentación de destino.

Para definir un **proyecto de documentación** simplemente debemos indicar el tipo de salida que deseemos, el directorio de 
[plantillas de generación de documentación](/blog/applications/nsharpdoc/plantillas-de-nsharpdoc/plantillas-de-nsharpdoc) y 
el directorio de salida de la documentación final.

Una vez seleccionado, podemos añadir los elementos que deseamos documentar en la lista de la ficha de proveedores utilizando la barra
de herramientas:

![Pantalla de definición de un proveedor de documentación para NSharpDoc](/blog/applications/nsharpdoc/pantalla-proveedor.jpg "Pantalla de definición de un proveedor de documentación")

Actualmente existen tres tipos de proveedores:


* **C#:** permite la documentación de código fuente escrito en C#. En este caso podemos seleccionar la solución o el
	proyecto que deseamos documentar.
* **Visual Base.Net:** documenta código fuente escrito en Visual Basic.Net. Del mismo modo que en el caso anterior
	podemos seleccionar el proyecto o solución a documentar.
* **SQL Server:** documenta el esquema de base de datos SQL Server. Para ello debemos indicar los datos de la base de datos
	(servidor, esquema y usuario) que deseemos documentar.
* **Estructuras de documentación:** archivos XML de documentación. Explicaremos que es este tipo de documentación más adelante.

El directorio de documentación adicional de esta ventana le indica a la aplicación dónde se encuentran los archivos adicionales (esta
versión aún no utiliza esta información).

Por supuesto, podemos definir tantos proveedores como necesitemos y de los tipos adecuados:

![Proyecto con proveedores definidos de NSharDoc](/blog/applications/nsharpdoc/pantalla-principal-proyecto.jpg "Proyecto con proveedores definidos")

Una vez definidos los proveedores de nuestro proyecto podemos grabarlo para volver a utilizarlo más adelante y pulsar el botón **Procesar**
para generar la documentación. 

## Documentación generada

Al generar la documentación del proyecto que hemos configurado en la pantalla principal, se crean una serie de páginas HTML de acuerdo con la
[plantilla](/blog/applications/nsharpdoc/plantillas-de-nsharpdoc/plantillas-de-nsharpdoc) seleccionada. Para el ejemplo, he generado
documentación de un proyecto en C#, un par de bases de datos y un par de estructuras XML. La página principal es
la siguiente:

![Indice de la documentación generada por NSharpDoc](/blog/applications/nsharpdoc/indice-documentacion.jpg "Indice de la documentación")

Donde vemos los diferentes proyectos y las bases de datos. Si seleccionamos uno de los proyectos, nos aparecerá
el índice principal del proyecto con la lista de espacios de nombres y clases:

![Indice de la documentación de un proyecto](/blog/applications/nsharpdoc/documentacion-proyecto.jpg "Indice de la documentación")

y si pulsamos sobre una de ellas llegaremos a la documentación específica de la clase:

![Documentación de clase](/blog/applications/nsharpdoc/documentacion-clase.jpg "Documentación de clase")

Lo mismo ocurre si en el índice de la documentación pulsamos sobre una base de datos:

![Indice de documentación de base datos](/blog/applications/nsharpdoc/documentacion-base-datos.jpg "Documentación de base datos")

y posteriormente seleccionamos la documentación de una tabla:

![Documentación de tabla de base de datos](/blog/applications/nsharpdoc/documentacion-tabla.jpg "Documentación de tabla de base de datos")

Por supuesto, el aspecto de la documentación es completamente configurable modificando las 
[plantillas de NSharpDoc](/blog/applications/nsharpdoc/plantillas-de-nsharpdoc/plantillas-de-nsharpdoc).

## Funcionamiento interno de la aplicación

Aunque intento explicar la forma de extender **NSharpDoc** mediante diferentes 
[proveedores](/blog/applications/nsharpdoc/proveedores-de-nsharpdoc) en otra sección, creo que es importante 
añadir aquí parte de esta información como base.

Como todos los desarrolladores saben, existen una gran cantidad de aplicaciones destinadas a la gestión de la documentación
de código fuente pero muchas de ellas se basan en un entorno en concreto o simplemente en el código fuente y no permiten
documentar otros aspectos del desarrollo como las bases de datos o los requisitos.

En **NSharpDoc** el planteamiento es diferente, al comenzar con las primeras pruebas del artículo 
[documentación de código C# utilizando Roslyn](/blog/articles/source-code/documentacion-codigo-csharp-plantillas/documentacion-codigo-csharp-plantillas)
y añadirle plantillas de generación me di cuenta que las estructuras de documentación tenían un aspecto similar independientemente
de su origen, es decir, podía hacer una aplicación que utilizase plantillas para documentar estructuras de datos
independientemente de si estos datos venían de proyectos de código fuente o de una base de datos e incluso de otras 
fuentes siempre que tuviesen una estructura normalizada.

Esto permite facilitar todo el proceso: a partir de diferentes entradas (código fuente, base de datos...) generar
estructuras de documentación normalizadas y a partir de estas estructuras normalizadas generar documentación.
Quizá quede más claro si vemos un esquema de estructura de la aplicación:

![Esquema de la aplicación NSharpDoc](/blog/applications/nsharpdoc/esquema-aplicacion.jpg "Esquema de la aplicación")

En este diagrama, los cuadros rojos identifican los [proveedores](/blog/applications/nsharpdoc/proveedores-de-nsharpdoc), 
es decir, las aplicaciones encargadas de interpretar código fuente o esquemas de bases de datos o estructuras de documentación 
en XML. Estas aplicaciones leen los datos de origen, código fuente o esquema de base de datos, y generan un 
objeto de estructura de documentación.

Un objeto de [estructura de documentación](/blog/applications/nsharpdoc/estructuras-de-documentacion)
mantiene la estructura jerárquica de la fuente. Es decir, una aplicación tiene espacios
de nombres que contienen clases que contienen métodos; una base de datos por su parte contiene tablas que contienen columnas. 
Ambos tipos se pueden mantener con una estructura recursiva en forma de árbol.

Por supuesto, estas estructuras también contienen parámetros, por ejemplo, una columna tiene un parámetro que identifica si admite
o no nulos, un método tiene un parámetro con el nombre y otro indicando si es asíncrono o no...

Una vez que el intérprete inicial ha generado las estructuras de documentación el gestor de plantillas se encarga de leer las 
plantillas y añadirle la información de las estructuras.

Esta mezcla entre plantillas y estructuras de documentación se le envía al generador que escribe los documentos finales, concretamente,
en el caso de HTML generaría el sitio Web final con la documentación.

Las estructuras de documentación XML son un caso específico de documentación que permite extender la aplicación sin necesidad de
programación. En realidad a **NSharpDoc** no le importa de dónde hayan salido estos archivos para poder generar la documentación
siempre y cuando tengan la estructura adecuada (que se explica en la sección
[estructuras de documentación](/blog/applications/nsharpdoc/estructuras-de-documentacion). Este
tipo de archivos nos permiten extender **NSharpDoc** de una forma sencilla con otros lenguajes o sistemas de bases de datos 
o fuentes diversas sin necesidad de modificar el motor de gestión de plantillas.

## Descarga de la aplicación

El código fuente de la aplicación se puede descargar de [GitHub](https://github.com/jbautistam/NSharpDoc).