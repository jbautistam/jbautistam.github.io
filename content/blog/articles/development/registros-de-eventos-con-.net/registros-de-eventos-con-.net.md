+++
title = "Registros de eventos con .NET"
date = "2019-10-11"
description = "Manejo del registro de eventos utilizando C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Sistema" ]
+++

Uno de los asuntos pendientes en el desarrollo de la aplicación suele ser la información que proporcionamos
sobre el funcionamiento de la aplicación tanto para los usuarios como para los departamentos de sistema.

Si nuestra aplicación tiene algún problema ¿cómo sabe el usuario o el responsable de sistemas qué es lo que
se estaba haciendo y dónde ha fallado? Si no se conoce esta información es muy complicado diagnosticar y 
corregir el problema.

Uno de los lugares más útiles para registrar los errores y el funcionamiento de nuestra aplicación es
el **Registro de Eventos** de Windows (accesible desde las **Herramientas administrativas** desde
el **Panel de Control** ). Aquí se debería indicar información detallada que nos permita
saber exactamente lo que está pasando y recuperarnos del problema.

La verdad es que la utilización del **registro de eventos** desde.NET es bastante sencillo. Una vez
importada la librería de acceso al registro de eventos ( #i System.Diagnostics # ) Sólo tenemos
que hacer dos cosas:

* Registrar una rama de aplicación en el **registro de sucesos**.
* Añadir nuestro **registros de log** sobre la rama de la aplicación.

## Creación de un registro de eventos

Para crear el **registro de eventos** podemos utilizar este método

```csharp
public static void CreateEventSoure(string strProject, string strLogName)
{ 
	if (!EventLog.SourceExists(Project))
		EventLog.CreateEventSource(Project, LogName);
}
```

## Escritura de un evento sobre el registro

Para enviar una línea de log al **registro de eventos** podemos utilizar el siguiente método:

```csharp
public static void EventLogEntryType(string strProject, EventLogEntryType intTypeLog, string strMessage)
{ 
	EventLog.WriteEntry(Project, strMessage, intTypeLogSystem);
}
```

Donde **EventLogEntryType** es un enumerado con el tipo de mensaje que estamos añadiendo al **registro de sucesos**. 
En este enumerado podemos indicar tres valores:

* `EventLogEntryType.Information`: línea de registro informativa.
* `EventLogEntryType.Warning`: línea de registro con una advertencia de la aplicación.
* `EventLogEntryType.Error`: línea de registro con un error de la aplicación.