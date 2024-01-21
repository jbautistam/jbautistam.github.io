+++
title = "Conversión imágenes a Base64 y viceversa en C#"
date = "2019-10-11"
description = "Código en C# para la conversión de imágenes a cadenas en Base64 y de cadenas en Base64 a imágenes"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación" ]
+++

En ocasiones, por ejemplo cuando se va a enviar una imagen en un archivo XML, debemos codificar una imagen
utilizando **Base64**.

El proceso de conversión de la imagen a una cadena en **Base64** es bastante sencillo.

Simplemente debemos grabar la imagen sobre un stream en memoria, convertirlo a bytes y posteriormente
codificarlo en **Base64** utilizando la función #i ToBase64String # del objeto #i Convert #:

```csharp
/// <summary>
///	  Codifica una imagen en Base64
/// </summary>
public string EncodeImage(Image objImage, System.Drawing.Imaging.ImageFormat intFormat)
{ 
	byte[] arrBytImages;
	
	// Abre un stream en memoria para obtener los bytes de la imagen		
	using (System.IO.MemoryStream stmMemory = new System.IO.MemoryStream())
	{ 
		// Graba la imagen en el stream en memoria
		objImage.Save(stmMemory, intFormat);
		// Convierte la imagen en un array de bytes
		arrBytImages = stmMemory.ToArray();
		// Cierra el stream
		stmMemory.Close();
	}
	// Devuelve el array de bytes convertidos
	return Convert.ToBase64String(arrBytImages);
}
```

El proceso inverso para traducir una cadena codificada en `Base64` a una imagen es igualmente sencillo, pero
al contrario, es decir, obtenemos el array de bytes a partir de la cadena (utilizando la función `FromBase64String`
del objeto `Convert`), lo escribimos en memoria y lo leemos en una imagen utilizando el método
estático `Image.FromStream`:

```csharp
/// <summary>
///		Obtiene una imagen a partir de la codificación en Base64
/// </summary>
public Image GetImage(string strEncoded)
{ 
	byte[] arrBytImage = Convert.FromBase64String(strEncoded);
	Image objImage = null;
		
	// Convierte un array de bytes a una imagen
	using (System.IO.MemoryStream stmBitmap = new System.IO.MemoryStream(arrBytImage, 0, arrBytImage.Length))
	{ 
		// Escribe el array de bytes en el stream de memoria
		stmBitmap.Write(arrBytImage, 0, arrBytImage.Length);
	  	// Carga la imagen del stream
		objImage = Image.FromStream(stmBitmap);
	  	// Cierra el stream
		stmBitmap.Close();
	}
	// Devuelve la imagen
	return objImage;
}
```
