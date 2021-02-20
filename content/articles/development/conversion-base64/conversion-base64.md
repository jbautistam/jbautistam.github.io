+++
title = "Codificación de archivos en Base64 C#"
date = "2019-10-11"
description = "Cómo codificar archivos en Base64 en el lenguage C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Sistema" ]
+++

Codificar una cadena en **Base64** utilizando CSharp es un proceso bastante sencillo.

Simplemente debemos utilizar la función **ToBase64String** de la clase **Convert**.

Por ejemplo, para codificar el contenido de un archivo en Base64 utilizamos este
código:

```csharp
private string Encode(string strFileTarget)
{ 
	string strEncoded;

	using (FileStream fs = new FileStream(strFileTarget, FileMode.Open, FileAccess.Read))
	{ 
		byte[] filebytes = new byte[fs.Length];
	
			fs.Read(filebytes, 0, Convert.ToInt32(fs.Length));
			strEncoded = Convert.ToBase64String(filebytes, Base64FormattingOptions.InsertLineBreaks);
	}
	return strEncoded;
}
</pre>
```

A este método, le pasamos como parámetro el archivo que deseamos codificar y nos devuelve la
cadena codificada en Base64.

Decodificar una cadena en Base64 a un array de bytes es un proceso igualmente sencillo aunque
en este caso utilizamos la función **FromBase64** de la clase **Convert**.

Si por ejemplo, deseamos grabar los datos codificados como una cadena en Base64 en un archivo decodificado,
podemos utilizar este código:

```csharp
private void Decode(string strFileTarget, string strEncoded)
{ 
	byte[] arrBytFilebytes = Convert.FromBase64String(strEncoded);

	using (FileStream stmFile = new FileStream(strFileTarget, FileMode.CreateNew, FileAccess.Write, FileShare.None))
		{ 
			stmFile.Write(arrBytFilebytes, 0, arrBytFilebytes.Length); 
			stmFile.Close(); 
		}
}
}
```
