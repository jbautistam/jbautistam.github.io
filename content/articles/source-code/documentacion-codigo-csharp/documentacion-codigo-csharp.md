+++
title = "Documentación de código fuente de C#"
date = "2019-10-11"
description = "Versión preliminar de la aplicación para documentación de código fuente C# en HTML utilizando Roslyn"
thumbnail = "/articles/source-code/documentacion-codigo-csharp/documentacion-codigo-csharp.jpg"
tags = [ "Programación" ]
+++

Todo desarrollador sabe que la documentación de un proyecto es muy importante. Todo desarrollador sabe 
también que la documentación de un proyecto es la parte más aburrida de nuestro trabajo.

Por eso, para facilitar la documentación del proyecto, prácticamente todos los lenguajes de programación 
modernos nos permiten añadir la documentación básica al propio código fuente: Java utiliza los comentarios 
de Javadoc, Python los comentarios en función y los lenguajes de.Net los comentarios XML de cabecera. 
Incluso para lenguajes que no incorporan esta funcionalidad tenemos aplicaciones como 
[Doxygen](http://www.stack.nl/~dimitri/doxygen/) que nos permiten extraer la documentación a partir de comentarios 'normales'.

En el framework de.Net se incluye desde el principio de los tiempos la posibilidad de generar archivos XML 
con los comentarios especiales del código al compilar nuestra aplicación. El problema es que no incluye también 
una aplicación para generar una documentación legible en formatos como HTML o de archivos de ayuda. 

Por supuesto, podemos recurrir a aplicaciones externas como [Sandcastle](https://sandcastle.codeplex.com/) o la propia 
**Doxygen**. Estas aplicaciones parten del XML generado en la compilación y lo combinan con 
la información de los ensamblados utilizando Reflection. De esta forma obtienen la documentación final.

Esto no supone ningún problema, pero la llegada de [Roslyn](https://github.com/dotnet/roslyn) 
abre paso a nuevas herramientas que podemos  utilizar a la hora de documentar nuestro código. Como estamos viendo en la serie de artículos sobre 
[Roslyn](Artículos\2016\Roslyn 1) de esta misma Web, ahora disponemos de una API 
completa para acceder al código fuente y a la estructura de definición de clases de nuestros proyectos.

Aprovechando esta API podemos leer el código fuente de un proyecto o una solución, recorrer sus estructuras 
y generar la documentación. Esto es precisamente lo que intento con mi nuevo proyecto que, a falta de un 
nombre mejor, llamaremos **RoslynDoc** y cuyos fuentes podéis consultar en 
[GitHub](https://github.com/jbautistam/RoslynDoc): una aplicación para generación de documentación de código fuente en C# (por ahora).

## Estructura de la aplicación

La aplicación de documentación de C# con **Roslyn** se divide en tres proyectos:

* **RoslynDoc:** aplicación WinForms. Simplemente presenta el interface de usuario para seleccionar el 
proyecto o solución que vamos a documentar, el directorio de salida y los parámetros básicos como los 
tipos de archivos de salida y el ámbito de las estructuras que se van a incluir en la documentación.

* **LibRoslynManager:** librería de clases que se encarga de la interpretación del código fuente utilizando 
**Roslyn** y genera el árbol de clases que posteriormente se va a documentar.

* **LibRoslynDocument:** librería que a partir de la estructura de clases obtenida como salida de la librería 
anterior genera los archivos de documentación.

Como podemos imaginar, se ha dividido en dos librerías para que podamos reutilizar el código. Así por ejemplo, 
si no nos gusta la documentación generada por la segunda librería siempre podemos programar esta parte siguiendo 
otros criterios. También nos da la posibilidad de ampliar el proceso para incluir otro tipo de documentación 
como por ejemplo diagramas de clases.

De hecho, una de las posibles mejoras de la herramienta podría ser separar el modelo de clases de la librería 
**LibRoslynManager** del proceso de interpretación. Eso nos permitiría añadir librerías de interpretación, 
por ejemplo, de Visual Basic que generasen el modelo de clases intermedio. Dado que la documentación se 
compila a partir de la jerarquía de clases del modelo y no de las estructuras de **Roslyn** podríamos 
añadir tantos lenguajes de entrada como quisiéramos. Por supuesto, esto se deja en manos del lector.

Aparte, hay otros proyectos en la solución con controles para WinForms y librerías comunes que vamos a 
obviar en esta explicación.

### LibRoslynManager

La librería de interpretación de código tiene un único punto de entrada, la clase `ProgramParser` que nos 
ofrece diferentes métodos sobrecargados para documentar un proyecto o una solución o bien una clase suelta. 
En cualquier caso, si el proceso de interpretación es correcto, nos devuelve un objeto `ProgramModel` con la 
estuctura de clases interpretadas.

Esta clase se apoya en los métodos de la clase `FileParser` que es la encargada de interpretar los archivos 
de código fuente por separado llamando al método `ParseText` de la API de **Roslyn** que nos devuelve 
un árbol sintáctico y preparando el árbol semántico que vamos a utilizar:

```csharp
/// <summary>
///	Interpreta un texto
/// </summary>
private void ParseText(CompilationUnitModel objUnit, string strText)
{ 
	CSharpCompilation objCompilation;

	// Crea el modelo de compilación
	objCompilation = CSharpCompilation.Create("ParserText").AddSyntaxTrees(CSharpSyntaxTree.ParseText(strText));
	// Obtiene el árbol semántico
	objTreeSemantic = objCompilation.GetSemanticModel(objCompilation.SyntaxTrees[0], true);
	// Interpreta los nodos
	ParseNodes(objUnit, objTreeSemantic.SyntaxTree.GetRoot());
}
```

Recorriendo este árbol sintáctico podemos generar nuestra estructura de clases. Para ello, buscamos los nodos 
que nos interesan, es decir los nodos de declaración de espacios de nombres, clases, estructuras, enumerados, 
métodos...

```csharp
/// <summary>
///	Interpreta los hijos de un nodo
/// </summary>
private void ParseChilds(SyntaxNode objRoot, LanguageStructModel objParent)
{ 
	foreach (SyntaxNode objNode in objRoot.ChildNodes())
		ParseNode(objNode, objParent);
}

/// <summary>
///	Interpreta los datos de un nodo
/// </summary>
private void ParseNode(SyntaxNode objNode, LanguageStructModel objParent)
{ 
	switch (objNode.Kind())
	{ 
		case SyntaxKind.NamespaceDeclaration:
			ParseNameSpace(objNode, objParent);
			break;
		case SyntaxKind.ClassDeclaration:
		  	ParseClasses(objNode, objParent);
		  	break;
	  	
	  	......
	}
}
```

Una vez localizado un nodo interesante buscamos en el árbol semántico la información adicional sobre ese nodo. 
Este nodo semántico proporciona información sobre la declaración: en una clase nos dice el tipo base, en un 
método los agumentos y el valor de retorno, etc... Así sólo tenemos que transformar este 
nodo semántico en una de las clases de modelo que posteriormente utilizaremos para documentar. Por ejemplo, 
una clase se transforma con un método similar a este:

```csharp
/// <summary>
///	Interpreta una clase
/// </summary>
private void ParseClass(SyntaxNode objNode, LanguageStructModel objParent)
{	
	ClassModel objClass = objParent.Items.CreateClass(objParent);
	INamedTypeSymbol objSymbol = objTreeSemantic.GetDeclaredSymbol(objNode as ClassDeclarationSyntax);

	// Obtiene el nombre y los comentarios
	objStructItem.Name = objSymbol.Name;
	// Obtiene las propiedades de la clase
	objClass.IsStatic = objSymbol.IsStatic;
	objClass.IsSealed = objSymbol.IsSealed;
	objClass.IsAbstract = objSymbol.IsAbstract;
	// Obtiene los métodos, propiedades y demás
	ParseChilds(objNode, objComplex);
	..........
}
```

Como se puede apreciar del código anterior, todas las clases de modelo derivan de la clase `LanguageStructModel`
que aparte de los datos básicos como nombre, tipo y espacio de nombres incluyen una colección de tipo 
`LanguageStructModel`. Es decir, se trata de una estructura recursiva. Dentro de un objeto de espacio 
de nombres podremos tener interfaces, clases, estructuras... O dentro de una clase podremos tener otras clases 
o métodos o propiedades. Esta estructura en árbol imita la estructura del propio árbol sintáctico de **Roslyn** 
aunque eliminando gran parte de su complejidad. Por supuesto tenemos además clases específicas como `ClassModel`
o `MethodModel` que derivan de `LanguageStructModel` pero con propiedades propias de ese tipo de datos para 
facilitar su documentación.

Para terminar, algo curioso de los árboles sintácticos de **Roslyn** es que incluyen lo que denominan 'nodos 
trivia'. Estos nodos contienen por ejemplo espacios en blanco, paréntesis y comentarios que normalmente se 
desechan en la compilación. En el caso que nos ocupa además nos ofrecen algo muy interesante: el XML de los 
comentarios de documentación. De hecho, en la última versión ya no es necesario recorrer los nodos trivia, 
podemos recogerlos directamente utilizando el método `GetDocumentationCommentXml()` de los nodos semánticos:

```csharp
/// <summary>
///	Inicializa los datos de la estructura
/// </summary>
private void AssignRemarksXML(LanguageStructModel objStructItem, ISymbol objSymbol)
{ 
	if (objSymbol != null)
		objStructItem.RemarksXml.RawXml = objSymbol.GetDocumentationCommentXml();
}
```

Con este último añadido al terminar el proceso de interpretación tenemos ya nuestra estructura jerárquica de 
espacios de nombres, clases, métodos, etc... que podemos pasar a la siguiente librería para generar la documentación.

### LibRoslynDocument

Esta librería es la encargada de recoger la estructura de clases de `ProgramModel` y generar los archivos 
finales de documentación. 

Actualmente permite tres tipos de archivos: HTML, XML y Nhaml (un lenguaje de plantilla similar al HTML que 
utilizo en mi aplicación de generación de sitios Web 
[BauDocWriter](http://bauplugstudio.webs-interesantes.es/Plugins/BauDocWriter/BauDocWriter.htm)
con la que se crea por ejemplo este sitio Web).

Para hacer la librería más versatil, puede recibir un objeto que implemente la interface `IDocumentWriter` quien 
se encargará de generar los archivos finales y que nos serviría por ejemplo para procesar otros tipos de 
archivos como PDF o DocBook o archivos de ayuda de Windows.

Para permitir este tipo de extensiones, la librería no genera archivos finales sino estructuras de nodos y 
atributos que posteriormente las clases que implementan `IDocumentWriter` pueden interpretar y grabar los 
archivos en el formato adecuado. Esta estructura de nodos ya es básicamente el documento final, por ejemplo, 
para una clase podemos obtener esta estructura en árbol de nodos de la clase `MLNode`:

* h1
	* Clase Class1
* h2
	* Métodos
	* table
		* tr
			* th
				* Ambito
			* th
				* Nombre
		* tr
			* td
				* public
			* td
				* Save

Para ver cómo se tratan estos nodos podemos consultar por ejemplo la clase `HtmlWriter`.

Por último, entre los parámetros de documentación se incluye el modo de generación, este modo de generación 
indica cómo se van a crear y separar los archivos. Por ejemplo, el modo complejo crea un archivo por separado 
por cada espacio de nombres, clase, método, porpiedad, etc... El modo simple crea menos archivos porque en
el archivo de documentación de una clase se incluye la información de todos sus métodos y propiedades. Es decir, 
el segundo modo crea una documentación bastante más compacta que el primero.

Por supuesto la idea es que ese modo de generación se transforme también en un interface y se pueda inyectar en 
el método para generar diferentes tipos de documentación o que se transforme completamente para utilizar 
plantillas en lugar de código puro y duro.

## El futuro

Esta herramienta, como se puede ver en el código, aún está en una fase inicial pero ya permite la generación de 
la documentación básica aunque le falten cosas como los estilos. Sin embargo, pensando en el futuro y profundizando 
un poco más en todas las posibilidades que nos ofrece **Roslyn** podríamos añadir funcionalidades para generar 
gráficos de clases y de interconexiones entre las clases o incluso tablas de llamadas entre métodos o métricas de código.
	
Precisamente la gran ventaja de **Roslyn** es que nos da toda la información sobre el código de nuestras aplicaciones 
sin necesidad de trabajo de interpretación adicional.

Supongo por tanto que iré modificando la aplicación para añadirle más funcionalidades. Si alguien quiere sumarse 
al proyecto, como comentaba al principio del artículo, podéis encontrar todos los fuentes en 
[GitHub](https://github.com/jbautistam/RoslynDoc).