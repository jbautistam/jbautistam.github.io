+++
title = "Encriptar cadenas utilizando Triple DES"
date = "2019-10-11"
description = "Forma de encriptar una cadena utilizando el algoritmo Triple DES en C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

**DES** es un algoritmo de encriptación en bloque de clave simétrica, es decir, se utiliza la misma clave para
encriptar como para desencriptar un mensaje.
	
Encriptar un mensaje utilizando **DES** con las librerías de.NET (`System.Security.Cryptography^) 
es bastante sencillo, prácticamente se reduce a seleccionar el proveedor específico y llamar 
a los métodos adecuados.

## Inicializando el proveedor criptográfico

El primer paso antes de cifrar o descifrar un mensaje es inicializar el proveedor **TripleDES** con la clave:

```csharp
private static TripleDES GetInstance(string strKey)
{ 
	TripleDES objProvider = new TripleDESCryptoServiceProvider();
	
	// Inicializa el proveedor
	objProvider.Key = Encoding.Unicode.GetBytes(strKey);
	objProvider.IV = new byte[objProvider.BlockSize / 8];
	// Devuelve el proveedor
	return objProvider;
}
```

## Encriptación de un mensaje

El método de encriptación de una cadena en un array de bytes es:

```csharp
public static byte[] Encrypt(string strMessage, string strKey)
{ 	
	TripleDES objProvider = GetInstance(strKey);
	ICryptoTransform objCrypto = objProvider.CreateEncryptor();
	byte[] arrBytBuffer = Encoding.Unicode.GetBytes(strMessage);
	
	// Devuelve el arry de bytes encriptado
	return objCrypto.TransformFinalBlock(arrBytBuffer, 0, arrBytBuffer.Length);
}
```

Dado que lo que se nos devuelve es un array de bytes, podemos pasarlo a una cadena. Dado que este array
de bytes está encriptado y por tanto puede contener caracters no imprimibles, normalmente al pasarlo a una
cadena se utiliza una codificación en base 64:
	
```csharp
public static string EncryptToBase64(string strMessage, string strKey)
{ 	
	return Convert.ToBase64String(Encrypt(strMessage, strKey))
}
```	

## Desencriptar una cadena

Para desencriptar una cadena la función es igualmente sencilla (el método espera que el mensaje sea
una cadena codificada en base 64, por eso la primera línea es la instrucción `Convert.FromBase64String`):

```csharp
public static string Decrypt(string strMessage, string strKey)
{	
	byte[] arrBytBuffer = Convert.FromBase64String(strMessage);
	TripleDES objProvider = GetInstance(strKey);
	ICryptoTransform objCrypto = objProvider.CreateDecryptor(); 
	
	// Devuelve la cadena desencriptada
	return Encoding.Unicode.GetString(objCrypto.TransformFinalBlock(arrBytBuffer, 0, arrBytBuffer.Length));
}
```

## Vulnerabilidades

La mayor vulnerabilidad de los algoritmos de clave simétrica está en la clave. Si la clave es demasiado corta
puede que sea fácil descifrarlo, sin embargo, los usuarios no son demasiado amigos de utilizar claves de más
de 8 caracteres.

Para evitar utilizar en el cifrado una clave demasiado corta se puede emplear en lugar de la clave proporcionada
por el usuario un [hash MD5](/blog/articles/seguridad/obtener-el-valor-md5-de-una-cadena/obtener-el-valor-md5-de-una-cadena)
de la clave lo que nos dará una cadena más larga que podemos usar con mayor confianza.