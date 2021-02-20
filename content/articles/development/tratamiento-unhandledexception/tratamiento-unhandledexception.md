+++
title = "Tratamiento de la excepción AppDomain.CurrentDomain.UnhandledException en Windows Forms y WPF"
date = "2019-10-11"
description = "Cómo se deben tratar las excepciones de  AppDomain.CurrentDomain.UnhandledException tanto en Windows Forms como en WPF"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Sistema" ]
+++

Cuando programamos una aplicación, normalmente tratamos las excepciones más o menos habituales en nuestros
métodos, sin embargo, hay excepciones, que bien por su rareza o bien porque nunca se nos ha dado en las
pruebas de desarrollo, no tenemos localizadas.

En estos casos, cuando las aplicaciones **Windows Form** o **WPF** encuentran una **excepción no controlada**
simplemente finalizan la ejecución del programa mostrando un error al usuario (bastante críptico por cierto).

Por supuesto, existen formas de capturar esas **excepciones no controladas** para que no le aparezcan al usuario
o bien les aparezca un mensaje más amigable. También podemos crear métodos que manejen esa excepción y nos envíen
por correo información de log de lo que estaba haciendo el usuario en ese momento para poder corregir el error
en futuras versiones de la aplicación.

Para conseguirlo, debemos tratar el evento **AppDomain.CurrentDomain.UnhandledException** desde nuestra clase
**Program** o similar, es decir, la clase de entrada a nuestra aplicación. Para un mayor control de todas las
**excepciones no controladas**, también podemos tratar el evento **Application.ThreadException** que se
lanza cuando ocurre algún error dentro del hilo principal.

El código es bastante sencillo, simplemente debemos escribir las siguientes líneas en el método **Main** (el método de 
inicio de la aplicación marcado con el atributo **STAThread**:

```csharp
/// <summary>
///		Punto de entrada a la aplicación
/// </summary>
[STAThread]
private static void Main(string [] arrStrFileNames)
{	
	// Captura las excepciones de aplicación
	Application.ThreadException += new System.Threading.ThreadExceptionEventHandler(Application_ThreadException);
	// Añade el manejador de eventos para tratar las excepciones de procesos no relacionados con la interface
	AppDomain.CurrentDomain.UnhandledException += new UnhandledExceptionEventHandler(CurrentDomain_UnhandledException);
	// Configura el modo de tratamiento de las excepciones no tratadas para que todos los errores vayan a través
	// del manejador
	Application.SetUnhandledExceptionMode(UnhandledExceptionMode.CatchException);
	// Resto de métodos para inicializar la aplicación
	.....
}

/// <summary>
///		Evento de tratamiento de excepciones en la aplicación
/// </summary>
private static void Application_ThreadException(object sender, System.Threading.ThreadExceptionEventArgs e)
{	
	TreatError(null, "Excepción no controlada", e.Exception);
}

/// <summary>
///		Evento de tratamiento de excepciones en la aplicación
/// </summary>
private static void CurrentDomain_UnhandledException(object sender, UnhandledExceptionEventArgs e)
{ 
	TreatError(null, "Excepción no controlada", e.ExceptionObject as Exception);
}
```

En el método `TreatError` pondremos las instrucciones necesarias para tratar el error, puede ser desde
una grabación de log o el envío de correos al departamento de sistemas o simplemente mostrarle un mensaje al
usuario con la descripción de la excepción.

Este método depende realmente de nuestra aplicación por eso no lo mostramos aquí, eso sí, deberíamos procurar que
fuese una rutina lo más rápida posible para que no interrumpa demasiado el trabajo del interface de usuario. 