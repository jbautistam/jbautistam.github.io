+++
title = "Información del método llamante en C# 4.5"
date = "2019-10-11"
description = "Cómo obtener información del método que ha llamado a otro en C# 4.5"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación" ]
+++

En ocasiones, cuando se desarrollan aplicaciones avanzadas en.NET deseamos tener información sobre el
método que nos está llamando.

Uno de los ejemplos más claros son los sistemas de traza y depuración, pero puede que lo debamos utilizar
en otros casos.

En C# 4.5 se ha añadido una serie de atributos que podemos utilizar en el método llamado para recoger
información sobre el método llamante.

Estos atributos se encuentran en el espacio de nombres **System.Runtime.CompilerServices** y lo podemos
utilizar en cualquier método aunque la estructura es ligeramente extraña ya que se deben añadir como
parámetros opcionales en el método.

Quizá lo veamos más claramente con un ejemplo:

```csharp
private void TestInfo([CallerMemberName] string strSourceMemberName = "", 
					  [CallerFilePath] string strSourceFilePath = "", 
					  [CallerLineNumber] int intSourceLineNo = 0)
{ 
	Debug.WriteLine("Método llamante: " + strSourceMemberName);
	Debug.WriteLine("Archivo: " + strSourceFilePath);
	Debug.WriteLine("Nº de línea: " + intSourceLineNo);
}
```

En el método anterior, estamos utilizando tres argumentos con los atributos apropiados que.NET rellenará con
los datos del método llamante. Por supuesto, no es necesario inicializarlos en la llamada al método:

```csharp
private void Caller()
{ 
	TestInfo();
}
```

Podemos utilizar los siguientes atributos como argumentos:

* `CallerMemberName`: cadena con el nombre del método que realiza la llamada (los contructores
envía la cadena `.ctor` mientras que los destructores utilizan la cadena `Finalizer`).
* `CallerFilePath`: cadena con el nombre de archivo fuente donde se encuentra el método que realiza
la llamada.
* `CallerLineNumber`: número de línea en el archivo fuente en que se encuentra el método llamante.
