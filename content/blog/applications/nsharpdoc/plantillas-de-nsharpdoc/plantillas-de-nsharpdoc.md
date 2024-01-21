+++
title = "Plantillas de NSharpDoc"
date = "2019-10-11"
description = "Plantillas utilizadas para la generación de documentación en NSharpDoc"
thumbnail = "/applications/nsharpdoc/plantillas-de-nsharpdoc/directorio-plantillas.jpg"
tags = [ "Aplicaciones" ]
+++

[nsharpdoc.md](C:/Users/jbautistam/Proyectos/WebSites/Hugo/test-hugo/content/applications/nsharpdoc/nsharpdoc)
incorpora un sistema de plantillas que nos permite
personalizar completamente la documentación generada.

Uno de los principios básicos de este sistema de plantillas es que resulte fácil de implementar, por eso
utiliza HTML como base para el sistema de definición de páginas y una serie de archivos
XML que identifican las plantillas utilizadas en cada caso.

**NSharpDoc** incluye un directorio de plantillas básico que se encuentra en el subdirectorio 
*/Data/Templates/Main* de la aplicación principal. En la siguiente imagen podemos ver la 
lista de archivos de este directorio: 

![Directorio de plantillas de NSharpDoc](/blog/applications/nsharpdoc/plantillas-de-nsharpdoc/directorio-plantillas.jpg "Directorio de plantillas")
	
Por supuesto, podemos generar nuestras propias plantillas en cualquier directorio. Simplemente se requieren cuatro 
archivos básicos:

1. El archivo que identifica los grupos de plantillas, en la imagen **Templates.tpt**, que indica la 
plantilla que vamos a utilizar para cada tipo de estructura.

2. El archivo de índice **IndexTemplate.tpt** que nos sirve para definir las páginas de índice.

3. La plantilla de cada tipo de estructura (aunque varios tipos pueden compartir la misma plantilla), en la imagen los archivos
**ClassTemplate.tpt**, **DataBaseTemplate.tpt** o **ProcedureTemplate.tpt**, por ejemplo.

4. La plantilla global de la página donde se inserta el contenido de la documentación, en este caso **RootTemplate.tpt**.

Además, en el directorio aparecen otros archivos como **Test.htm** o los subdirectorios *Images* o *Styles* que contienen
imágenes, estilos o archivos JavaScript que simplemente se van a copiar al sitio web de salida.

Entre estos archivos, el único que tiene un nombre fijo es **Templates.tpt** dado que es el que identifica el 
resto de plantillas, el resto de archivos los podemos llamar como queramos siempre que tengan extensión **tpt** 
para que no se copien en el directorio de salida y los identifiquemos correctamente en este archivo de grupo.

A continuación explicaremos el contenido de cada archivo.

## Grupo de plantillas (archivo Templates.tpt)

En el directorio raíz donde definimos nuestras plantillas debe existir obligatoriamente un archivo llamado **Templates.tpt** 
que identifica los diferentes archivos de plantillas utilizadas para cada tipo de estructura. 

Para que la aplicación sepa las plantillas a utilizar, en el archivo *Templates.tpt* debemos identificar
el archivo de plantillas y el archivo raíz de cada tipo de estructura. El archivo de plantilla
incluido de ejemplo con **NSharpDoc** es el siguiente:
	
```XML
<Templates>
	<!-- Indice -->
	<Index Name ="Index" StructType = "Index" File ="IndexTemplate.tpt" RootTemplate="RootTemplate.tpt" />
	<!-- Plantillas para proyectos de código fuente -->
	<Page StructType ="Program" File = "ProgramTemplate.tpt" RootTemplate="RootTemplate.tpt" />
	<Page StructType ="NameSpace" File = "ClassTemplate.tpt" RootTemplate="RootTemplate.tpt" />
	<Page StructType ="Class" File = "ClassTemplate.tpt" RootTemplate="RootTemplate.tpt" />
	<Page StructType ="Interface" File = "ClassTemplate.tpt" RootTemplate="RootTemplate.tpt" />
	<Page StructType ="Struct" File = "ClassTemplate.tpt" RootTemplate="RootTemplate.tpt" />
	<!-- Plantillas para proyectos de base de datos -->
	<Page StructType ="DataBase" File ="DataBaseTemplate.tpt" RootTemplate ="RootTemplate.tpt" />
	<Page StructType ="Table" File ="TableTemplate.tpt" RootTemplate ="RootTemplate.tpt" />
	<Page StructType ="View" File ="TableTemplate.tpt" RootTemplate ="RootTemplate.tpt" />
	<Page StructType ="Procedure" File ="ProcedureTemplate.tpt" RootTemplate ="RootTemplate.tpt" />
	<Page StructType ="Function" File ="ProcedureTemplate.tpt" RootTemplate ="RootTemplate.tpt" />
</Templates>
```

En este archivo XML hay dos tipos de etiquetas. Las etiquetas **Index** identifican las plantillas que vamos a utilizar
en la páginas de índice mientras que las etiquetas **Page** identifican las plantillas utilizadas en cada tipo de estructura.

En las etiquetas *Page* se definen las plantillas de cada tipo, indicando en
*StructType* el tipo de estructura (espacio de nombres, clase, interface, estructura...) al que se 
corresponde la página y en *File* el archivo de plantilla que vamos a utilizar.

Debemos tener en cuenta que **NSharpDoc** es un sistema de documentación de código un tanto especial puesto que no se limita
a documentar código fuente si no que nos permite documentar cualquier tipo de información estructurada como se explica
en la página [estructuras de documentación](/blog/applications/nsharpdoc/estructuras-de-documentacion).

El atributo *RootTemplate* indica la plantilla raíz, es decir la plantilla común para todas las páginas de
salida (por supuesto, podríamos definir tantos archivos de plantilla raíz como quisiéramos aunque en el caso
de ejemplo siempre es el mismo). En el caso de HTML sería en esta plantilla en la que definiríamos las cabeceras y los pies
de las páginas. 

## Plantilla raíz

La plantilla raíz que definimos en el atributo *RootTemplate* del archivo *Templates.tpt* es la que identifica el
aspecto global del archivo de salida, es decir, contiene las cabeceras, estilos, referencias de javascript y demás de nuestra
página en el caso de HTML.	

En el caso del ejemplo, el archivo *RootTemplate.tpt* que se encuentra en el directorio **Main** es el siguiente:

```XML
<Page>
	<![CDATA[
		<html>
			<head>
				<title>{{Title}}</title>
				<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
				<link rel="stylesheet" href=" Styles/layout.css" >
			</head>
			<body>
				<div class="container">
					{{Body}}
				</div>
				<footer class = "container" >
					<nav>
						<ul class="pager">
							<li><a href=" Indice.htm">Indice</a></li>
							<li>{{TopPage}}</li>
						</ul>
					</nav>
					<div class="alert alert-success" role="alert">
						<p>
							Documentación generada con <a href="http://bauplugstudio.webs-interesantes.es/Plugins/BauDocWriter/BauDocWriter.htm">BauDocWriter</a>
						</p>
					</div>
				</footer>
			</body>
		</html>
	]]>
</Page>
```

Como podemos ver, en el nodo *Page* se incluye la información de la página en HTML (porque estamos generando
plantillas para sitios Web escritos en HTML). La aplicación sustituye los valores que aparecen entre llaves dobles (por ejemplo `{{Body}})` por
el resultado de la generación del documento.

En estas plantillas podemos utilizar los siguientes parámetros:

* **Title:** título del archivo
* **Description:** descripción del archivo
* **Body:** cuerpo con la documentación de la salida de las plantillas que veremos en el siguiente punto.
* **TopPage:** enlace a la página superior, es decir, al elemento padre en la estructura de archivos. Si por ejemplo
estamos en una clase el *TopPage* sería el documento del espacio de nombres.

## Plantillas de estructuras

Por supuesto, la más importante es la plantilla de estructura que es la que nos define cómo será la documentación de nuestro código fuente.

Para que todo sea más sencillo, la plantilla básicamente es código HTML con algunas particularidades y nodos especiales que dirigen el proceso
de documentación.

Además, al tratarse como un archivo XML debemos recordar que nuestro HTML debe estar bien formado, es decir, en los casos en que combinemos
párrafos o celdas con vínculos debemos utilizar nodos span.

Antes de explicarlo, veamos un ejemplo parcial de la plantilla *ClassTemplate.tpt*:

```XML
<Page>
	<div class="panel panel-primary">
		<div class="panel-heading">
			<h1 class="panel-title">{{StructType}} {{Name}}</h1>
		</div>
		<div class="panel-body">
			<ul>
				<li>
					<strong>Espacio de nombres:</strong>
					<span>{{NameSpace|Link}}</span>
				</li>
				<IfValue ValueType="BaseType">
					<li>
						<strong>Base:</strong>
						<span>{{BaseType|Link}}</span>
					</li>
				</IfValue>
			</ul>
			<p>{{Summary}}</p>
		</div>
	</div>

	<IfValue ValueType="Remarks">
		<div class="bs-callout bs-callout-info">
			<h4>Comentarios</h4>
			<p>{{Remarks}}</p>
		</div>
	</IfValue>

	<IfExists StructType = "Method">
		<hr/>
		<div class="panel panel-success">
			<div class="panel-heading">
				<h3 class="panel-title">Métodos</h3>
			</div>
			<div class="panel-body">
				<table class="table table-hover">
					<thead>
						<tr>
							<th>Ambito</th>
							<th>Nombre</th>
							<th>Descripción</th>
						</tr>
					</thead>
					<tbody>
						<ForEach StructType = "Method">
							<tr>
								<td>{{Modifier}}</td>
								<td>{{Name}}</td>
								<td>{{Summary}}</td>
							</tr>
							<tr>
								<td></td>
								<td colspan = "2">
									<p>{{Remarks}}</p>
									<p>{{Prototype}}</p>
									<br/>
									<IfExists StructType = "Argument">
										<h3>Parámetros</h3>
										<ul>
											<ForEach StructType = "Argument">
												<li>
													<Part IsBold ="yes">{{Name}} ({{Type|Link}}):</Part>
													<Part>{{Summary}}</Part>
												</li>
											</ForEach>
										</ul>
										<br/>
									</IfExists>
									<IfExists StructType = "ReturnValue">
										<h3>Valor de retorno</h3>
										<Select StructType ="ReturnValue">
											<p>
												<Part IsBold="true">Tipo:</Part>
												<span>{{Type|Link}}</span>
											</p>
											<p>{{Summary}}</p>
										</Select>
									</IfExists>
								</td>
							</tr>
						</ForEach>
					</tbody>
				</table>
			</div>
		</div>
	</IfExists>
</Page>
```

En la plantilla existen algunos nodos especiales que no se corresponden con el HTML. Estos nodos son:


* **IfValue:** indicamos que sólo se debe insertar en el archivo final las etiquetas hijas de este nodo si hay algún
valor en uno de los campos de la estructura que estamos documentando. El campo que consultamos se indica en el atributo
*ValueType*. En el caso del ejemplo (*IfValue ValueType = 'Remarks'*) indicamos que sólo se escriba el contenido
del nodo si la estructura a documentar tiene algún valor en el campo **Remarks**, es decir, si tiene comentarios adicionales.

* **IfExists:** con este nodo indicamos que sólo se deben insertar en el archivo final las etiquetas hijas si alguno de los
elementos de la estructura es del tipo especificado en el atributo **StructType**. En el caso del ejemplo 
(*IfExists StructType = 'Method'*) indicamos que sólo se debe insertar si hay algún método definido en la estructura, es decir,
si estamos en una clase indica que se debe ejecutar cuando tiene métodos o funciones.

* **ForEach:** indica que por cada una de las estructuras del tipo indicado en **StructType** se deben incluir
los nodos hijo. En el caso de ejemplo, con *ForEach StructType = 'Method'*, definimos que se inserten las etiquetas HTML
que haya dentro del nodo **ForEach** por cada uno de los métodos de la estructura.

* **Select:** este último nodo es similar al **ForEach** anterior pero en este caso sólo se recoge una estructura.

Del mismo modo que en la plantilla anterior, indicamos entre llaves dobles, los textos que debemos insertar de cada estructura. Por
ejemplo con `{{Name}}` apuntamos al nombre de la estructura que se está documentando en cada momento.

Existen casos especiales cuando queremos añadir un vínculo. Para ésto, añadimos un carácter de pipeline y la etiqueta *Link*. Por
ejemplo con `{{ReturnType|Link}}` indicamos que se escriba un vínculo al tipo del valor de retorno de un método si se encuentra
entre los documentados, por ejemplo, *Int32* no aparece como vínculo porque no está definido entre los tipos del proyecto documentado
pero *Class1* incluye el vínculo hacia el documento de la clase *Class1* (por supuesto, si está en el proyecto que documentamos).

Además de estas etiquetas tenemos otra etiqueta especialmente diseñada para transformar el contenido de la salida de la documentación. Para
explicarlo vamos a utilizar una sección de la plantilla **TableTemplate.tpt**:
	
```XML
<ForEach StructType ="Column">
	<tr>
		<td>
			<Switch ValueType="IsIdentity">
				<Case Value="True">
					<img src="##Images/PrimaryKey.png##" alt="Clave principal"/>
				</Case>
			</Switch>
		</td>
		<td>
			<Switch ValueType="IsForeignKey">
				<Case Value="True">
					<img src="##Images/ForeignKey.png##" alt="Clave foránea"/>
				</Case>
			</Switch>
		</td>
		<td>{{Name}}</td>
		<td>{{Summary}}</td>
		<td>{{Type}}</td>
		<td>
			<Switch ValueType="IsNullable">
				<Case Value="True">
					<span>Sí</span>
				</Case>
				<Case Value="False">
					<span>No</span>
				</Case>
			</Switch>
		</td>
	</tr>
</ForEach>
```

Los nuevos nodos corresponden con las etiquetas **Switch** que nos permiten cambiar el valor que viene de la documentación por el que nos interese para
nuestra documentación. Por ejemplo, esta sección de la plantilla está documentando las columnas de una tabla gracias al nodo #b ForEach StructType = 'Column' #
y presentando cada una de las columnas en una tabla. La siguiente estructura muestra una imagen cuando tenemos una columna de clave primaria:

```XML
<Switch ValueType="IsIdentity">
	<Case Value="True">
		<img src="##Images/PrimaryKey.png##" alt="Clave principal"/>
	</Case>
</Switch>
```

El atributo **ValueType** nos indica el campo del que consultamos el valor y las etiquetas **Case** en el atributo **Value** identifican los
diferentes valores posible, en este caso, cuando en la estructura de columna, tengamos un valor **True** en la propiedad **IsIdentity** vamos
a insertar la imagen definida en la etiqueta **img**.

Los caracteres ## que aparecen en el atributo **src** de la imagen, son para que **NSharpDoc** transforme la URL de la imagen en una URL
relativa a la página en el sitio Web.

Por supuesto en la etiqueta **Switch** podemos añadir varios nodos **Case** e incluso un nodo **Default** que se inserta cuando no se encuentra ningún
valor:

```XML
<Switch ValueType="IsNullable">
	<Case Value="True">
		<span>Sí</span>
	</Case>
	<Case Value="False">
		<span>No</span>
	</Case>
	<Default>
		<span>No definido</span>
	</Default>
</Switch>
```

## Plantilla de índice

El último archivo de plantilla es **Index.tpt** que define las páginas de tabla de contenido o índice.

En el caso de ejemplo, la plantilla tiene este contenido:

```XML
<Page>
	<IfExists StructType = "Program">
		<div class="panel panel-primary">
			<div class="panel-heading">
				<h1 class="panel-title">Proyectos</h1>
			</div>
			<div class="panel-body">
				<ul>
					<ForEach StructType ="Program">
						<li>{{Name|Link}}</li>
					</ForEach>
				</ul>
			</div>
		</div>
	</IfExists>

	<IfExists StructType ="DataBase">
		<div class="panel panel-primary">
			<div class="panel-heading">
				<h1 class="panel-title">Bases de datos</h1>
			</div>
			<div class="panel-body">
				<ul>
					<ForEach StructType ="DataBase">
						<li>{{Name|Link}}</li>
					</ForEach>
				</ul>
			</div>
		</div>
	</IfExists>
</Page>
```

Con dos secciones, una que lista las estructuras de tipo **Program** con la documentación de código fuente y otra
que lista las estructuras de tipo **DataBase** con la documentación de base de datos.