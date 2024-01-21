+++
title = "Encriptación utilizando WDP con C#"
date = "2019-10-11"
description = "Métodos de encriptación utilizando WDP (Windows Data Protection) con C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

**Windows Data Protection (WDP)** es un sistema de encriptación que proporciona el propio sistema operativo utilizando una clave generada
automáticamente a partir de la contraseña del usuario que ha iniciado la sesión. Por ejemplo, este es el sistema empleado cuando se utilizan
los métodos del espacio de nombres `System.IO.File.Encrypt` sobre un archivo.

Si deseamos utilizar el mismo sistema desde nuestras aplicaciones podemos utilizar la clase `ProtectedData` del espacio de nombres
`System.Security.Cryptography`. Esta clase permite también utilizar una clave global para todos los usuarios de una máquina.
	
Por supuesto, la fortaleza de la encriptación **WDP** depende de la contraseña del usuario aunque se puede añadir una mayor seguridad
añadiéndole datos aleatorios y pasándoselos al método de encriptación.

El siguiente ejemplo muestra cómo utilizar las funciones para encriptar y desencriptar una cadena:

```csharp
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

class Test
{
	static void Main()
	{ 
		string strFileName = "Cifrado.txt"; 
	    string strSource = "Esta es la cadena que se va a encriptar";
	    string strTarget;
	    byte[] arrBytEntropy = new byte[] { 281, 293, 11, 1, 29, 1920 }; //... bytes aleatorios para aumentar la seguridad
	
			// Encripta el texto y lo almacena en el archivo
			Encrypt(strText, strFileName, DataProtectionScope.LocalMachine, arrBytEnthropy);
			
			// Mensajes al usuario
			Console.WriteLine("Texto encriptado en el archivo");
			Console.ReadLine();
			
	    // Desencripta el texto de un archivo
	    strTarget = Decrypt(strFileName, DataProtectionScope.LocalMachine, arrBytEnthropy);
	        
	    // Mensajes para el usuario
	    Console.WriteLine("Texto desencriptado: " + strTarget);
	    Console.ReadLine();
  }
    
  /// <summary>
  ///   Encripta la cadena en un archivo
  /// </summary>
  private static void Encrypt(string strText, string strFileName, DataProtectionScope intScope, byte [] arrBytEnthropy)
  { 
  	byte[] arrBytSource = new UTF8Encoding().GetBytes(strText); //... pasa la cadena original a bytes
	byte[] arrBytEncrypted = ProtectedData.Protect(arrBytSource, arrBytEntropy, intScope); //... array de bytes codificados
      
    // Escribe en el archivo
    File.WriteAllBytes(strFileName, arrBytEncrypted);
  }

	/// <summary>
	///   Descifra los datos del archivo
	/// </summary>
	private static void Encrypt(string strFileName, DataProtectionScope intScope, byte [] arrBytEnthropy)
	{ 
		byte[] arrBytFile = File.ReadAllBytes(strFileName);
		byte[] arrBytDecrypted = ProtectedData.Unprotect(arrBytFile, arrBytEntropy, intScope);

		// Devuelve la cadena decodificada
		return new UTF8Encoding().GetString(arrBytDecrypted);
	}
}
```