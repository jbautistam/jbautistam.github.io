+++
title = "Obtener el valor SHA1 de una cadena en C#"
date = "2019-10-11"
description = "Cómo obtener el valor de la función Hash SHA1 de una cadena en C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

El algoritmo **SHA1**, del mismo modo que el algoritmo **MD5**, otiene una cadena única utilizando una
[función de Hashing](http://en.wikipedia.org/wiki/Hashing_function) sobre una cadena de entrada.
	
Para conseguir una cadena **SHA1** en C# se puede utilizar la siguiente clase estática:

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

namespace CiberGestion.Libraries.LibCryptography.Hashing
{
	/// <summary>
	///		Clase para codificación SHA1
	/// </summary>
	public static class SHA1Helper
	{
		/// <summary>
		///		Calcula una cadena SHA1 a partir de una cadena de entrada
		/// </summary>
		public static string Compute(string strSource)
		{ 
			byte [] arrBytTarget;
			SHA1 objSHA1 = new SHA1CryptoServiceProvider();
			
				// Codifica la cadena
				arrBytTarget = objSHA1.ComputeHash(ASCIIEncoding.Default.GetBytes(strSource));
				// Convierte los bytes codificados en una cadena legible
				return BitConverter.ToString(arrBytTarget).Replace("-", "");
		}
	}
}
```

Para obtener el resultado **SHA1** para una cadena simplemente debemos realizar una llamada a la función:

```csharp
string strHashValue = SHA1Helper.Compute("Hello World");
```

Los lectores fieles y observadores, se habrán dado cuenta que la forma de trabajar es prácticamente
la misma que en el artículo que explicaba como 
[obtener el valor MD5 de una cadena](/blog/articles/seguridad/obtener-el-valor-md5-de-una-cadena/obtener-el-valor-md5-de-una-cadena),
en realidad lo único que cambia es el proveedor criptográfico utilizado, en el caso de **SHA1** utilizamos
la clase `SHA1CryptoServiceProvider`, mientras que para el caso de **MD5** utilizamos la
clase `MD5CryptoServiceProvider`. 
