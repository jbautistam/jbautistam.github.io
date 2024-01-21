+++
title = "Compresión URL con TinyURL mediante C#"
date = "2019-10-11"
description = "Forma de obtener una URL comprimida utilizando C# y el servicio TinyUrl"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación" ]
+++

A una de las cosas que ha contribuido **Twitter** en los últimos años ha sido a popularizar los servicios de
compresión de URL como [TinyUrl](http://tinyurl.com).

Estos servicios se encargan de recoger una Url larga como `http://www.urllarga.com/mes/articulo/muy/aburrido.html` y
devolver una Url mucho más corta (como `http://tiniurl.com/u2322` ) adecuada para los 140 caracteres máximos
de **Twitter**.

Aparte de poder utilizar el servicio manualmente, la mayoría de estos acortadores ofrecen la opción de hacerlo
automáticamente. Así en **TinyUrl** debemos llamar a la página #i api-create.php # con un parámetro indicándole la 
Url que deseamos acortar y nos devuelve una cadena con la Url corta.

Un ejemplo con la Url anterior sería: `http://tinyurl.com/api-create.php?url=http://www.urllarga.com/mes/articulo/muy/aburrido.html`,
si lo escribís en la dirección del navegador os aparecerá como resultado una única línea con la Url comprimida.

Por supuesto podemos utilizar las funciones de `HttpWebRequest` y `HttpWebResponse` para hacer lo mismo mediante código:

```csharp
public static string CompressURL(string strURL)
{ 
	HttpWebRequest objRequest = WebRequest.Create("http://tinyurl.com/api-create.php?url=" + strURL);
	string strResponse = null;
	
	try
	{ 
		HttpWebResponse objResponse = objRequest.GetResponse() as HttpWebResponse;
		StreamReader stmReader = new StreamReader(objResponse.GetResponseStream());
		
		strResponse = stmReader.ReadToEnd();			
	}
	catch {}
	return strResponse;
}
```

La mayor parte de los servicios de este tipo funcionan de forma similar, además proporcionan un enlace
a la **API** que explica como hacerlo aunque desafortunadamente no hay un método estándar para todos. 
Por lo tanto habrá que adaptar el código anterior para el servicio de compresión y ya tendremos nuestra rutina
lista.