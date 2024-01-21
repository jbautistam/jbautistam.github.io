+++
title = "Obtener las aplicaciones instaladas en Windows con C#"
date = "2019-10-11"
description = "Artículo que explica cómo obtener las aplicaciones instaladas en un ordenador con Windows con una aplicación en C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Sistema" ]
+++

En ocasiones resulta muy útil saber qué aplicaciones tenemos instaladas en el ordenador bien porque deseamos ver incompabilidades o crear
un log extenso en caso de error o simplemente porque deseemos crear un inventario de éstas.

Podemos saber las aplicaciones instaladas en un ordenador de dos formas: utilizando el registro o 
[WMI](http://msdn.microsoft.com/en-us/library/aa394582(v=vs.85).aspx) (Windows Management Instrumentation).

En este artículo utilizaremos **WMI** simplemente por perderle el miedo a esta tecnología y porque (para mi gusto) resulta bastante más sencillo.

**WMI** es una tecnología de Microsoft que lleva presente en nuestras máquinas practicamente desde Windows 2000 y ofrece un API sencillo
de acceso prácticamente a cualquier información de sistema a partir de consultas similares al SQL o mediante enumerados.

Para acceder a información de **WMI** desde nuestra aplicación.NET debemos utilizar el espacio de nombres `System.Management`
que ofrece cuatro clases:

* `ManagementObject` o `ManagementClass`: que presenta un objeto o clase de administración.
* `ManagementObjectSearcher`: que permite consultar una colección de `ManagementObject` o `ManagementClass` a partir de
una consulta SQL o un enumerado.
* `ManagementEventWatcher`: que se utilizapara suscribirse a notificaciones de eventos de **WMI**.
* `ManagementQuery`: utilizado como base para todas las clases de consulta.


En nuestro caso, para obtener la información sobre las aplicaciones instaladas en nuestro equipo debemos utilizar la clase
[Win32_Product](http://msdn.microsoft.com/en-us/library/aa394378%28v=vs.85%29.aspx)
que presenta información sobre los productos instalados utilizando el **Windows Installer**.	

La función es tan simple como realizar la consulta en SQL y recorrer la colección devuelta mostrando el nombre en la consola:

```csharp
private void ShowInstalledApplications()
{ 
	ManagementObjectSearcher objColProducts = new ManagementObjectSearcher("SELECT * FROM Win32_Product");  
	
	// Recorremos los elementos obtenidos mostrándolos en la consola
	foreach (ManagementObject objProduct in objColProducts.Get())
	Console.WriteLine(objProduct["Name"]);  
}
```

Si desea más información sobre cómo utilizar **WMI** desde el Framework de.NET nada mejor que este artículo en la
[MSDN](http://msdn.microsoft.com/en-us/library/aa720264.aspx).