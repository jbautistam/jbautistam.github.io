+++
title = "Cómo convertir una cadena a Unicode con caracteres de escape en C#"
date = "2019-10-11"
description = "Función para convertir una cadena a Unicode utilizando caracteres de escape en C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación" ]
+++

En algunas ocasiones es necesario convertir una cadena en otra cadena Unicode utilizando caracteres de escape.

Es decir, lo que queremos es convertir caracteres como la 'ñ' en su codificación equivalente en C# en formato
Unicode: '\u00f1'.

La forma de hacerlo es bastante sencilla, simplemente debemos saber que todos los caracteres con un código mayor
a 127 se consideran caracteres fuera de la tabla ASCII y por tanto se deben codificar utilizando su valor
numérico en cuatro bytes en hexadecimal.

Aquí os dejo el código en forma de [método de extensión](Artículos\Programación\Métodos de extensión) (es
decir, debe ir en una clase estática e incluirla utilizando un **using** en las clases donde la queramos utilizar):

```csharp
/// <summary>
///		Codificar caracteres en Unicode
/// </summary>
public static string ToUnicode(this string strValue) 
{ 
	string strResult = "";

	// Codifica los caracteres
	foreach (char chrChar in strValue) 
	{ 
	if (chrChar > 127) 
		strResult += "\\u" + ((int) chrChar).ToString( "x4" );
	 else 
		strResult += chrChar;
	}
	// Devuelve el resultado
	return strResult;
}
```

Para utilizarlo simplemente debemos importar la clase con este **método de extensión** y utilizar una llamada de este tipo:

```csharp
string strConverted = "Prueba de conversión con ñ".ToUnicode();
```