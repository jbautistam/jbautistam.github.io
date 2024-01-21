+++
title = "Almacenamiento de datos sensibles con DPAPI"
date = "2019-10-11"
description = "Modo de almacenar datos sensibles como contraseñas utilizando DPAPI con C#"
thumbnail = "/articles/seguridad/dpapi-almacenamiento/dpapi-almacenamiento.jpg"
tags = [ "Programación", "Seguridad" ]
+++

En ocasiones, tenemos que almacenar las contraseñas u otros datos sensibles en un archivo o una base de datos
para utilizarlos posteriormente.
	
Para guardar estos datos de forma segura, podemos utilizar uno de los **sistemas criptográficos** existentes
como [AES](/blog/articles/seguridad/encriptacion-utilizando-rijndael-aes/encriptacion-utilizando-rijndael-aes) o 
[Triple DES](/blog/articles/seguridad/encriptar-cadenas-utilizando-triple-des/encriptar-cadenas-utilizando-triple-des). 

Nuestro problema es que estos sistemas criptográficos precisan una contraseña y de dónde obtener esta contraseña.
	
Si no se la podemos solicitar al usuario si no que debe asociarse a la aplicación, una forma sencilla es
escribirla directamente en el código. Esto, por supuesto, nos dará problemas en cuanto alguien decodifique
nuestro código y obtenga la cadena.
	
Sería bastante mejor si realmente el propio sistema operativo nos proporcionase una clave única para
almacenar datos en ese equipo. Afortunadamente, existe desde Windows 2000, se llama 
[Data Protection application programming interface (DPAPI)](https://msdn.microsoft.com/en-us/library/ms995355.aspx)
y es bastante sencilla de utilizar.
		
De hecho, sólo tiene dos funciones, una para encriptar y la otra para desencriptar:

```csharp
public byte[] Encrypt(string text)
{
	byte[] buffer;

		// La longitud de la cadena debe ser múltiplo de 16
		while (text.Length % 16 != 0)
			text += " ";
		// Convierte el texto a un array de bytes para encriptarlo
		buffer = Encoding.Unicode.GetBytes(text);
		// Encripta los datos en memoria, el resultado se almacena en el mismo buffer que los datos originales
		ProtectedMemory.Protect(buffer, MemoryProtectionScope.SameLogon);
		// Devuelve la cadena encriptada
		return buffer;
}

public string Decrypt(byte [] encrypted)
{
	// Desencripta los datos en memoria
	ProtectedMemory.Unprotect(encrypted, MemoryProtectionScope.SameLogon);
	// Devuelve la cadena desencriptada
	return Encoding.Unicode.GetString(encrypted);
}
```

Lo único *especial* de este código es que el texto debe tener una longitud múltiplo de 16, por eso le 
añadimos espacios al final de la cadena. Realmente, el número de bytes del buffer que utilizamos para
encriptar debe ser múltiplo de 16, por eso utilizamos un `Encoding.Unicode` para transformar la
cadena en un array de bytes.
	
Como vemos, en ningún caso nos solicita contraseña para encriptar o desencriptar, simplemente le indicamos 
el ámbito del que debe recogerla, en este ejemplo utiliza una contraseña asociada al usuario 
(`MemoryProtectionScope.SameLogon`).
	
Por supuesto, también tenemos funciones para encriptar / desencriptar un stream de datos:

```csharp
public int EncryptDataToStream(byte[] buffer, byte[] entropy, DataProtectionScope scope, Stream streamData)
{
	byte[] encrypted = ProtectedData.Protect(buffer, entropy, ConvertScope(scope));

		// Escribe los datos encriptados
		if (streamData.CanWrite && encrypted.Length > 0)
			streamData.Write(encrypted, 0, encrypted.Length);
		// Devuelve el número de datos escritos
		return encrypted.Length;
}

public byte[] DecryptDataFromStream(byte[] entropy, DataProtectionScope scope, Stream streamData, int length)
{
	byte[] inBuffer = new byte[length];
	byte[] outBuffer;

		// Lee los datos encriptados del stream y los desencripta
		if (streamData.CanRead)
		{
			streamData.Read(inBuffer, 0, length);
			outBuffer = ProtectedData.Unprotect(inBuffer, entropy, ConvertScope(scope));
		}
		else
			throw new IOException("No se puede leer del stream");
		// Devuelve los datos desencriptados
		return outBuffer;
}

public byte [] CreateRandomEntropy()
{
	byte[] entropy = new byte[16];

		// Rellena el array con un valor aleatorio (a partir de RNGCryptoServiceProvider)
		new RNGCryptoServiceProvider().GetBytes(entropy);
		// Devuelve el array
		return entropy;
}
```

Aunque es bastante sencillo, debemos tener en cuenta que para encriptar / desencriptar podemos utilizar un
valor de entropía, es decir, una serie de valores aleatorios que mejorarán la encriptación. Para
ellos podemos usar tanto la función de ejemplo `CreateRandomEntropy`, como cualquier otra.
	
En segundo lugar, debemos mantener aparte del stream de datos, la longitud de este stream para poder descifrarlo
posteriormente.