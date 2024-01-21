+++
title = "Encriptación RSA utilizando certificados digitales con .NET"
date = "2019-10-11"
description = "Forma de utilizar certificados digitales para encriptar o desencriptar un mensaje utilizando .NET y el algoritmo RSA"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

Cuando utilizamos [certificados digitales](/blog/articles/seguridad/certificados-digitales/certificados-digitales)
para encriptar un mensaje, normalmente vamos a utilizar métodos de **criptografía asimétrica**.

Lo primero que debemos hacer es obtener el **certificado** del
[almacén de certificados](/blog/articles/seguridad/almacenes-de-certificados-en-.net/almacenes-de-certificados-en-.net) 
de nuestro ordenador:

```csharp
private X509Certificate2 GetCertificate(string strThumbCertificate)
{ 
	X509Store x509Store = new X509Store(StoreLocation.LocalMachine);
	
	// Abre el almacén
	x509Store.Open(OpenFlags.ReadOnly);
	// Busca el certificado adecuado
	foreach (X509Certificate2 storeCertificate in x509Store.Certificates)
		if (strThumbCertificate.Equals(storeCertificate.Thumbprint, 
									   StringComparison.CurrentCultureIgnoreCase))
			return storeCertificate;
	// Si ha llegado hasta aquí es porque no ha encontrado nada
	return null;
}
```

Para **encriptar** los datos deberíamos utilizar la **clave pública** del certificado del destinatario de
la siguiente forma:

```csharp
public string Encrypt(string strMessage, string strThumbCertificate)
{ 
	byte [] arrBytBuffer = Encoding.Unicode.GetBytes(strMessage);
	X509Certificate2 objCertificate = GetCertificate(strThumbCertificate);
	RSACryptoServiceProvider rsaProvider = (RSACryptoServiceProvider) objCertificate.PublicKey.Key;

		return Convert.ToBase64String(rsaProvider.Encrypt(arrBytBuffer, false));
}
```

Mientras que para **desencriptar** los datos deberíamos utilizar la **clave privada** de nuestro
**certificado** (el remitente debe haberlo encriptado utilizando nuestra clave pública):
	
```csharp
public string Decrypt(string strMessage, string strThumbCertificate)
{	
	X509Certificate2 objCertificate = GetCertificate(strThumbCertificate);
	RSACryptoServiceProvider objProvider = (RSACryptoServiceProvider) objCertificate.PrivateKey;

		return Encoding.ASCII.GetString(objProvider.Decrypt(Convert.FromBase64String(strMessage), false));
}
```
