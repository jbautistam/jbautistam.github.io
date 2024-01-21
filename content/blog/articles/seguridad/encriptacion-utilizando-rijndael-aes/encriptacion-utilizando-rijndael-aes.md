+++
title = "Encriptación utilizando Rijndael - AES con .NET"
date = "2019-10-11"
description = "Explicación sobre la forma de encriptar cadenas utilizando el algoritmo Rijndael en C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

Uno de los algoritmos de [criptografía asimétrica](/blog/articles/seguridad/clave-publica/clave-publica)
es el algoritmo [Rijndael](http://en.wikipedia.org/wiki/Rijndael)  o **AES**.
	
Utilizar las librerías de encriptación de.NET (`System.Security.Cryptography`) para encriptar o 
desencriptar una cadena utilizando **Rijndael** es igual de sencillo que utilizar el 
[algoritmo Triple DES](/blog/articles/seguridad/encriptar-cadenas-utilizando-triple-des/encriptar-cadenas-utilizando-triple-des)
simplemente debemos seguir los mismos pasos:

## Inicializando el proveedor criptográfico

El primer paso antes de encriptar o desencriptar un mensaje es inicializar el proveedor utilizando la
clave del usuario:

```csharp
private static RijndaelManaged GetInstance(string strKey)
{ 
	RijndaelManaged objProvider = new RijndaelManaged();
	
	// Inicializa el proveedor
	objProvider.Key = Encoding.Unicode.GetBytes(strKey);
	objProvider.IV = new byte[objProvider.BlockSize / 8];
	// Devuelve el proveedor
	return objProvider;
}
```

## Encriptación de un mensaje

Para encriptar una cadena obteniendo el resultado como un array de bytes podemos utilizar la siguiente rutina:

```csharp
public static byte [] Encrypt(string strMessage, string strKey)
{ 
	RijndaelManaged objProvider = GetInstance(strKey);
	ICryptoTransform objCrypto = objProvider.CreateEncryptor();
	byte [] arrBytBuffer = Encoding.Unicode.GetBytes(strMessage);
	
	// Devuelve el aray de bytes encriptado
	return objCrypto.TransformFinalBlock(arrBytBuffer, 0, arrBytBuffer.Length);
}
</pre>
```

Si deseamos una cadena codificada en base 64 (por ejemplo para grabarlo en un archivo de texto o un
nodo XML) podemos utilizar la siguiente rutina:
	
```csharp
public static string EncryptToBase64(string strMessage, string strKey)
{ 
	return Convert.ToBase64String(Encrypt(strMessage, strKey));
}
```	

## Desencriptar una cadena

Desencriptar una cadena es igualmente sencillo, inicializamos el proveedor y obtenemos una instancia para
desencriptar (`ICryptoTransform`), llamamos a la función `TransformFinalBlock` para obtener una
cadena de bytes desencriptada:

```csharp
public static string Decrypt(string strMessage, string strKey)
{ 
	byte [] arrBytBuffer = Convert.FromBase64String(strMessage);
	RijndaelManaged objProvider = GetInstance(strKey);
	ICryptoTransform objCrypto = objProvider.CreateDecryptor();

	// Devuelve la cadena desencriptada
	return Encoding.Unicode.GetString(objCrypto.TransformFinalBlock(arrBytBuffer, 0,
									  arrBytBuffer.Length));
}
```

La rutina anterior espera una cadena codificada en **base 64**, por eso la primer instrucción de la
rutina convierte ese texto en un array de bytes (`Convert.FromBase64String`).