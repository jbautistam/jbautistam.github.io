+++
title = "Documentación de código CSharp utilizando plantillas"
date = "2019-10-11"
description = "Aplicación de documentación de código fuente en CSharp utilizando plantillas"
thumbnail = "/articles/source-code/documentacion-codigo-csharp-plantillas/documentacion-codigo-csharp-plantillas.jpg"
tags = [ "Programación" ]
+++

En la parte final del artículo anterior sobre 
[documentación de código fuente en CSharp](/blog/articles/source-code/documentacion-codigo-csharp/documentacion-codigo-csharp)
comentaba que había dos formas de generar la documentación o bien con un método que generaba archivos
por separado en clases, interfaces y métodos o bien otro en el que los archivos se generaban con los
métodos dentro de las clases.
	
Al mirar el código de generación de estos archivos me dí cuenta que era bastante complicado generar
una documentación configurable directamente desde el código así que me decidí a incorporar plantillas
para la generación de los archivos de forma que cualquiera se pudiese crear sus plantillas e incorporar
estilos, imágenes y demás.
	
Al final ha resultado que la generación de documentación utilizando plantillas resulta más cómoda y fácil
que generando el código manualmente.
	
Para conseguirlo en realidad sólo necesitamos tres archivos:

1. El archivo índice de las plantillas que vamos a utilizar para cada tipo de estructura.
2. La plantilla de cada tipo de estructura (aunque varios tipos pueden compartir la misma plantilla).
3. La plantilla global de la página donde se inserta el contenido de la documentación.

## Definición de plantillas

En la aplicación, al seleccionar la plantilla, en realidad seleccionamos un directorio de plantillas. En este
directorio podemos añadir tanto las plantillas con extensión `tpt` como todos los archivos que deseemos.
	
La aplicación copia todos los subdirectorios y los archivos que no acaban en `.tpt`
en el directorio de salida. Así podemos incluir archivos de estilo, javascript o imágenes fácilmente y configurar
en todo detalle los archivos generados.

### Grupo de plantillas

Para que la aplicación sepa las plantillas a utilizar, en el archivo `Templates.tpt` debemos identificar
el archivo de plantillas y el archivo raíz que vamos a utilizar en cada tipo de estructura. El ejemplo que estoy
utilizando es el siguiente:
	
```xml
<Templates>
	<Page StructType ="NameSpace" File = "ClassTemplate.tpt" RootTemplate="RootTemplate.tpt" />
	<Page StructType ="Class" File = "ClassTemplate.tpt" RootTemplate="RootTemplate.tpt" />
	<Page StructType ="Interface" File = "ClassTemplate.tpt" RootTemplate="RootTemplate.tpt" />
	<Page StructType ="Struct" File = "ClassTemplate.tpt" RootTemplate="RootTemplate.tpt" />
</Templates>
```

Para generar este archivo, en las etiquetas `Page` se definen las plantillas de cada tipo, indicando en
`StructType` el tipo de estructura (espacio de nombres, clase, interface, estructura...) al que se 
corresponde la página y en `File` el archivo de plantilla que vamos a utilizar.
	
El atributo `RootTemplate` indica la plantilla raíz, es decir la plantilla común para todas las páginas de
salida (por supuesto, podríamos definir tantos archivos de plantilla raíz como quisiéramos aunque en el caso
de ejemplo siempre es el mismo). En el caso de HTML sería en esta plantilla en la que definiríamos las cabeceras y los pies
de las páginas. 

### Plantilla raíz

La plantilla raíz que definimos en el atributo `RootTemplate` del archivo `Templates.tpt` es la que identifica el
aspecto global del archivo de salida, es decir, contiene las cabeceras, estilos, referencias de javascript y demás de nuestra
página en el caso de HTML.	

El archivo `RootTemplate.tpt` que he utilizado para mis pruebas es el siguiente:

```xml
<Page>
	<![CDATA[
		<html>
			<head>
				<title>{{Title}}< /title>
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
							<li>< a href=" Indice.htm">Indice</a></li>
							<li>{{TopPage}}</li>
						</ul>
					</nav>
					<div class="alert alert-success" role="alert">
						<p>
							Documentación generada con <a href="http://bauplugstudio.webs-interesantes.es/Plugins/BauDocWriter/BauDocWriter.htm">BauDocWriter</a>
						</p>
					< /div>
				< /footer>
			</body>
		</html>
	]]>
</Page>
```

Como podemos ver, en el nodo `Page` en este caso se incluye la información de la página en HTML (en este caso estamos generando
plantillas para archivos HTML). La aplicación sustituye los valores que aparecen entre llaves dobles (por ejemplo `{{Body}}`) por
el resultado de la generación del documento.
	
En estas plantillas podemos utilizar los siguientes parámetros:

* **Title:** título del archivo
* **Description:** descripción del archivo
* **Body:** cuerpo con la documentación de la salida de las plantillas que veremos en el siguiente punto.
* **TopPage:** enlace a la página superior, es decir, al elemento padre en la estructura de archivos. Si por ejemplo
estamos en una clase el `TopPage` sería el documento del espacio de nombres.

### Plantillas de estructuras

Por supuesto, la más importante es la plantilla de estructura que es la que nos define cómo será la documentación de nuestro código fuente.

Para que todo sea más sencillo, la plantilla básicamente es código HTML con algunas particularidades y nodos especiales que dirigen el proceso
de documentación.
	
Además, al tratarse como un archivo XML debemos recordar que nuestro HTML debe estar bien formado, es decir, en los casos en que combinemos
párrafos o celdas con vínculos debemos utilizar nodos span.
	
Antes de explicarlo, veamos un ejemplo parcial de la plantilla `ClassTemplate.tpt`:

```html
<Page>
	<div class="panel panel-primary">
		<div class="panel-heading">
			<h1 class="panel-title">{{StructType}} {{Name}}< /h1>
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
									<IfExists StructType = "Parameter">
										<h3>Parámetros</h3>
										<ul>
											<ForEach StructType = "Parameter">
												<li>
													<Part IsBold ="yes">{{Name}} ({{Type|Link}}):</Part>
													<Part>{{Summary}}</Part>
												</li>
											</ForEach>
										</ul>
										<br/>
									</IfExists>
									<h3>Valor de retorno</h3>
									<p>
										<Part IsBold="true">Tipo:</Part>
										<span>{{ReturnType|Link}}</span>
									</p>
									<p>{{ReturnRemarks}}</p>
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

* `IfValue`: indicamos que sólo se debe insertar en el archivo final las etiquetas hijas de este nodo si hay algún
valor en uno de los campos de la estructura que estamos documentando. El campo que consultamos se indica en el atributo
`ValueType`. En el caso del ejemplo (`IfValue ValueType = 'Remarks'`) indicamos que sólo se escriba el contenido
del nodo si la estructura a documentar tiene algún valor en el campo `Remarks`, es decir, si tiene comentarios adicionales.
* `IfExists`: con este nodo indicamos que sólo se deben insertar en el archivo final las etiquetas hijas si alguno de los
elementos de la estructura es del tipo especificado en el atributo `StructType`. En el caso del ejemplo 
(`IfExists StructType = 'Method'`) indicamos que sólo se debe insertar si hay algún método definido en la estructura, es decir,
si estamos en una clase indica que se debe ejecutar cuando tiene métodos o funciones.
* `ForEach`: este último nodo indica que por cada una de las estructuras del tipo indicado en `StructType` se deben incluir
los nodos hijo. En el caso de ejemplo, con `ForEach StructType = 'Method'`, definimos que se inserten los nodos por cada uno
de los métodos de la estructura.

Del mismo modo que en la plantilla anterior, indicamos entre llaves dobles, los textos que debemos insertar de cada estructura. Por
ejemplo con `{{Name}}` apuntamos al nombre de la estructura que se está documentando en cada momento.
	
Existen casos especiales cuando queremos añadir un vínculo. Para ésto, añadimos un carácter de pipeline y la etiqueta `Link`. Por
ejemplo con `{{ReturnType|Link}}` indicamos que se escriba un vínculo al tipo del valor de retorno de un método si se encuentra
entre los documentados, por ejemplo, `Int32` no aparece como vínculo porque no está definido entre los tipos del proyecto documentado
pero `Class1` incluye el vínculo hacia el documento de la clase `Class1` (por supuesto, si está en el proyecto que documentamos).

### Un ejemplo de documentación

Por ejemplo, aquí tenemos una imagen de la salida de la documentación para una clase:

![Ejemplo de salida de documentación](/blog/articles/source-code/documentacion-codigo-csharp-plantillas/sample.jpg "Ejemplo de salida")

## El futuro

El código fuente lo podéis encontrar en [GitHub](https://github.com/jbautistam/RoslynDoc).