+++
title = "Trazas de comunicaciones en C#"
date = "2019-10-11"
description = "Cómo ver la traza del sistema de comunicaciones en C#"
thumbnail = "/articles/development/trazas-comunicaciones/trazas-comunicaciones.jpg"
tags = [ "Comunicaciones" ]
+++

Estos días, he tenido que enfrentarme a una aplicación para transmitir archivos
por [sFTP](https://es.wikipedia.org/wiki/SSH_File_Transfer_Protocol).

El problema principal, aunque parezca lo contrario, no ha sido la implementación del protocolo en sí, si no
averiguar porqué fallaban ciertas rutinas de comunicaciones y saber qué estaba pasando por debajo, es decir,
qué intentaba hacer.NET cuando le pedía abrir un stream o cambiarlo por un stream SSL.

Una de las opciones era utilizar [Fiddler](http://www.telerik.com/fiddler) o
alguna herramienta similar para ver las comunicaciones entre mi máquina y el servidor pero no encontraba información
real sobre lo que estaba haciendo mal.

No sé muy bien cómo (suele pasarme), llegué a un
[artículo muy interesante](https://blogs.msdn.microsoft.com/dgorti/2005/09/18/using-system-net-tracing/)
en la MSDN sobre trazas de comunicaciones en una aplicación en.NET, es decir, sobre cómo se podía obtener un archivo de log 
de las rutinas por las que pasaba la aplicación mientras se establecen las comunicaciones. Justo lo que necesitaba.

Y de hecho, es bastante sencillo, simplemente hay que añadir ciertos parámetros al **app.config** de nuestra aplicación
de pruebas (en mi caso, una consola):

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
		</startup>
	<system.diagnostics>
		<trace autoflush="true" />
		<sources>
			<source name="System.Net">
				<listeners>
					<add name="MyTraceFile"/>
				</listeners>
			</source>
			<source name="System.Net.Sockets">
				<listeners>
					<add name="MyTraceFile"/>
				</listeners>
			</source>
		</sources>

		<sharedListeners>
			<add
				name="MyTraceFile"
				type="System.Diagnostics.TextWriterTraceListener"
				initializeData="System.Net.trace.log"  />
		</sharedListeners>
		<switches>
			<add name="System.Net" value="Verbose" />
			<add name="System.Net.Sockets" value="Verbose" />
		</switches>
	</system.diagnostics>
</configuration>
```

Una vez modificado el archivo de configuración simplemente tenemos que ejecutar la aplicación y nos genera automáticamente un archivo de log 
con el nombre **System.Net.Trace.Log** en el directorio de ejecución con toda la información de traza de las comunicaciones.