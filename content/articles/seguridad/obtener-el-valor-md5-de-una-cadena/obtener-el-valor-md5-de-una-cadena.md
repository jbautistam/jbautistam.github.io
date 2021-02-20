+++
title = "Obtener el valor MD5 de una cadena en C#"
date = "2019-10-11"
description = "Forma de obtener el valor de MD5 de una cadena utilizando las funciones de Hashing de .NET en C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

Las [funciones de Hashing](http://en.wikipedia.org/wiki/Hashing_function) 
son funciones de una dirección que a partir de un dato de entrada obtienen un dato de salida único.
	
Es decir, cualquier variación en los datos de entrada (aunque sea de únicamente un bit) da como resultado un valor de salida
diferente.
	
Esta característica hace que se utilicen estas funciones muy a menudo en tareas como la comparación de archivos,
las **firmas digitales** o la **encriptación de contraseñas** u **obtención de índices** en **tablas
Hash**.
	
Existen varios algoritmos para la obtención de cadenas a partir de **funciones Hash**, una de las más utilizadas
es el algoritmo **MD5**.
	
Para conseguir una cadena **MD5** en C# se puede utilizar la siguiente clase estática:

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

namespace Hashing
{
	/// <summary>
	///		Clase para codificación MD5
	/// </summary>
	public static class MD5Helper
	{
		/// <summary>
		///		Calcula una cadena MD5 a partir de una cadena de entrada
		/// </summary>
		public static string Compute(string strSource)
		{ 
			byte [] arrBytTarget;
			MD5 objMD5 = new MD5CryptoServiceProvider();
			
				// Codifica la cadena
				arrBytTarget = objMD5.ComputeHash(ASCIIEncoding.Default.GetBytes(strSource));
				// Convierte los bytes codificados en una cadena legible
				return BitConverter.ToString(arrBytTarget).Replace("-", "");
		}
	}
}
```

Para obtener el resultado **MD5** para una cadena simplemente debemos realizar una llamada a la función:

```csharp
	string strHashValue = MD5Helper.Compute("Hello World");
```