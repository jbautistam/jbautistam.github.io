+++
title = "Serializar / deserializar objetos en C#"
date = "2019-10-11"
description = "Cómo serializar y deserializar objetos con C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación" ]
+++

Los conceptos de **serialización** y **deserialización** son bastante comunes en.NET y hacen referencia a la posibilidad de
grabar y cargar objetos fácilmente utilizando ciertas características de.NET.

Para utilizar la serialización en.NET lo primero que debemos hacer es marcar nuestra clase como **Serializable** utilizando el
atributo *Serializable*

```csharp
[Serializable]
public class MyClass
{ 
	public int Width { get; set; }
	
	public int Height { get; set; }
	
	[NonSerialized] 
	public int Area 
	{ 
		get { return Width * Height; }
	}
}
```

En el ejemplo anterior, tenemos una clase que hemos llamado **MyClass** y la hemos marcado como **Serializable**, por tanto
todas sus propiedades se almacenarán cuando serialicemos objetos de este tipo. En el mismo ejemplo hemos marcado una de las propiedades
con el atributo `NonSerialized` lo que indica que no se debe almacenar este valor.

Una vez hemos marcado nuestra clase como `Serializable` lo único que nos queda es almacenar los objetos en archivos o memoria. 

El proceso es muy sencillo. Por ejemplo, para grabarlo en un archivo podemos utilizar el siguiente código:

```csharp
public void SerializeObject(object objObject, string strFileName)
{ 
	byte[] arrBytes = new byte[MAX_LENGTH];
	
	// Serializamos el objeto en un stream de memoria
	using (Stream stmMemory = new MemoryStream(arrBytes, true))
	{ 
		IFormatter objFormatter = new BinaryFormatter();
		
		// Serializamos el objeto en memoria
		objFormatter.Serialize(stmMemory, objObject);
		// Cerramos el stream en memoria
		stmMemory.Close();
	}
	// Grabamos los datos en un archivo
	using (Stream stmFile = new FileStream(strFileName, FileMode.Create, FileAccess.Write, FileShare.ReadWrite))
	{ 
		// Grabamos los datos
		stmFile.Write(arrBytes, 0, arrBytes.Length);
		// Cerramos el archivo
		stream.Close();
	}
}
```

En este caso hemos utilizado, aparte del #i stream # sobre el archivo un #i stream # en memoria simplemente por
demostrar lo sencillo que es serializar un objeto a la memoria.

El proceso inverso utilizado para cargar un objeto (deserializarlo) es igualmente simple:

```csharp
public object DeserializaObject(string strFileName)
{ 
	object objTarget = null;
	byte [] arrBytes = new byte[MAX_LENGTH];
	
	// Deserializa el objeto del archivo en un array en memoria
	using (Stream stmFile = new FileStream(path, FileMode.Open, FileAccess.Read, FileShare.ReadWrite))
	{ 
		// Carga los datos sobre el array de bytes
		stmFile.Read(arrBytes, 0, arrBytes.Length);
		// Cierra el archivo
		stmFile.Close();
	}
	// Convierte el array de bytes en el objeto inicial
	using (Stream stmMemory = new MemoryStream(arrBytes, true))
	{ 
		IFormatter objFormatter = new BinaryFormatter();
		
		// Deserializa el objet
		objTarget = (object) formatter.Deserialize(stmMemory);
		// Cierra el stream
		stmMemory.Close();
	}
	// Devuelve el objeto leído
	return objTarget
}
```