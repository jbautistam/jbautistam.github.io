+++
title = "Almacenes de certificados digitales en .NET"
date = "2019-10-11"
description = "Utilización de .NET para el tratamiento de certificados digitales"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

.NET nos permite tratar los certificados del mismo modo que implementa otras librerías con algoritmos criptográficos.
Las clases para el acceso a los certificados se encuentran en el espacio de nombres `System.Security.Cryptography.X509Certificates`
que implementa los métodos del estándar X.509 v3 (estándar de infraestructura de clave pública PKI – Public Key Infrastructure).
	
En este espacio de nombres existen diferentes clases que permiten operaciones para el mantenimiento de almacenes así como la 
importación, exportación, borrado, enumeración y recuperación de la información de los certificados.

Las clases más importantes son:


* `X509Store`: representa un almacén X.509 que es un catálogo físico donde se almacenan y administran los certificados. 
Existen varios almacenes integrados agrupados en dos ubicaciones: máquina local (contiene los certificados compartidos 
por todos los usuarios) y usuario actual (contiene los certificados específicos del usuario actual).
* `X509Certificate` y `X509Certificate2`: representan un certificado X.509.
* `X509Certificate2Collection`: representa una colección de objetos X509Certificate2.

## Obtención de un almacén

Los almacenes, como decíamos antes, son catálogos donde se guardan los certificados. Para abrir un almacén se debe utilizar 
su nombre y la ubicación.
	
El nombre del almacén puede ser una cadena o bien uno de los valores predefinidos indicados en el enumerado `StoreName`
con los siguientes valores:

* `AddressBook`: almacén para otros usuarios 
* `AuthRoot`: almacén para autoridades certificadoras. 
* `CertificateAuthority`: almacén para autoridades certificadoras intermedias 
* `Disallowed`: almacén para certificados revocados 
* `My`: almacén para certificados personales 
* `Root`: almacén para autoridades certificadoras raíz 
* `TrustedPeople`: almacén para personas o recursos de confianza 
* `TrustedPublisher`: almacén para publicadores de confianza

En el caso de la ubicación se utiliza el enumerado `StoreLocation` con dos posibles valores:

	
* `CurrentUser`: el almacén del usuario actual.
* `LocalMachine`: el almacén asignado a la máquina local cuyos certificados están disponibles para todos los usuarios.
	
Como ejemplo podríamos utilizar el siguiente código:

```csharp
/// <summary>
///		Obtiene un almacén
/// </summary>
public static X509Store GetStore(StoreName intStoreName, StoreLocation intStoreLocation)
{ 
	return new X509Store(intStoreName, intStoreLocation);
}

/// <summary>
///		Obtiene un almacén
/// </summary>
public static X509Store GetStore(string strName, StoreLocation intStoreLocation)
{ 
	return new X509Store(strName, intStoreLocation);
}

/// <summary>
///		Crea un almacén
/// </summary>
public static void CreateStore(string strName, StoreLocation intLocation)
{ 
	X509Store objStore = new X509Store(strName, intLocation);
	 
	// Abre el almacén para escritura
		objStore.Open(OpenFlags.ReadWrite);
	// Cierra el almacén
		objStore.Close();
}
```

## Listado de certificados

Una vez que tenemos un almacén podemos enumerar todos los certificados almacenados allí. Para ello simplemente debemos 
acceder a la colección de certificados (Certificates) del almacén y recoger su información.

Para obtener la colección de certificados de un almacén utilizamos un código similar a este:

```csharp
/// <summary>
///		Obtiene los certificados de un almacén
/// </summary>
public static X509Certificate2Collection GetCertificates(X509Store objStore)
{ 
	X509Certificate2Collection objColCertificates = null;

	// Abre el almacén para lectura
		objStore.Open(OpenFlags.ReadOnly);
	// Obtiene los certificados
		objColCertificates = objStore.Certificates;
	// Cierra el almacén
		objStore.Close();
	// Devuelve la colección de certificados
		return objColCertificates;
}
```
		
Una vez tenemos la colección de certificados podríamos mostrar los atributos de un certificado:

```csharp
/// <summary>
///		Imprime la información de una colección de certificados
/// </summary>
private void PrintInfo(X509Certificate2Collection objColCertificates)
{ 
	foreach (X509Certificate2 objCertificate in objColCertificates)
		PrintInfo(objCertificate);
}

/// <summary>
///		Imprime la información de un certificado
/// </summary>
private void PrintInfo(X509Certificate2 objCertificate)
{ 
	Console.WriteLine("Nombre: {0}", objCertificate.FriendlyName);
	Console.WriteLine("Emisor: {0}", objCertificate.IssuerName.Name);
	Console.WriteLine("Sujeto: {0}", objCertificate.SubjectName.Name);
	Console.WriteLine("Versión: {0}", objCertificate.Version);
	Console.WriteLine("Válido desde: {0}", objCertificate.NotBefore);
	Console.WriteLine("Válido hasta: {0}", objCertificate.NotAfter);
	Console.WriteLine("Número de serie: {0}", objCertificate.SerialNumber);
	Console.WriteLine("Algoritmo de firma: {0}", objCertificate.SignatureAlgorithm.FriendlyName);
	Console.WriteLine("Huella digital: {0}", objCertificate.Thumbprint);
	Console.WriteLine();
}
```

## Creación de un almacén

.NET también nos permite la creación de almacenes personalizados tanto para el usuario actual como para la máquina. Para 
ello lo único que debemos hacer es pasar a la función apropiada el nombre de un almacén que no exista y abrirlo con 
cualquiera de los valores de OpenFlags excepto OpenExistingOnly.

```csharp
/// <summary>
///		Crea un almacén
/// </summary>
public static void CreateStore(string strName, StoreLocation intLocation)
{ 
	X509Store objStore = new X509Store(strName, intLocation);
	 
	// Abre el almacén para escritura
	objStore.Open(OpenFlags.ReadWrite);
	// Cierra el almacén
	objStore.Close();
}
```

## Importación de certificados

Pero un almacén no nos serviría de nada si no pudiésemos añadirle certificados. Para importar un certificado 
se deben seguir los siguientes pasos:
	
* Crear un objeto `X509Certificate2` utilizando el directorio adecuado para el archivo de certificado
* Abrir el almacén donde vamos a importar con el derecho de acceso `ReadWrite`
* Añadir el certificado al almacén 
* Cerrar el almacén

El siguiente código nos sirve de ejemplo para importar un certificado:

```csharp
/// <summary>
///		Importa un certificado en un almacén
/// </summary>
public static void ImportCertificate(X509Certificate2 objCertificate, X509Store objStore)
{ 
	// Abre el almacén
	objStore.Open(OpenFlags.ReadWrite);
	// Añade el certificado
	objStore.Add(objCertificate);
	// Cierra el almacén
	objStore.Close();
}
```

## Exportación de certificados

Del mismo modo puede que deseemos grabar los datos de un certificado en un archivo. En la exportación de un certificado se 
deben seguir pasos similares a la importación:
	
* Abrir el almacén adecuado 
* Encontrar el certificado que se va a exportar
* Exportar el contenido del certificado a un stream de bytes
* Grabar los datos a un archivo
* Cerrar el almacén
	
Para buscar un certificado utilizamos el método `Find` de la colección de certificados. La búsqueda se puede realizar por 
diferentes criterios como la huella digital, el nombre del sujeto, el nombre del emisor, los periodos de validez, etc. 
Estas opciones de búsqueda se definen en el enumerado `X509FindType`.

En el siguiente ejemplo la búsqueda se realizará por nombre. Hay que tener en cuenta que en un almacén pueden existir 
varios certificados con el mismo nombre pero con diferentes números de serie. El ejemplo sólo exporta el primer 
certificado que encuentra en la colección.

```csharp
/// <summary>
///		Exporta un certificado a un archivo
/// </summary>
public static void ExportCertificate(string strCertificateName, string strFileName, X509Store objStore)
{ 
	X509Certificate2Collection objColCertificates;
	
	// Abre el almacén
	objStore.Open(OpenFlags.ReadOnly);
	// Busca los certificados que coinciden con el nombre
	objColCertificates = objStore.Certificates.Find(X509FindType.FindBySubjectName, strCertificateName, true);
	// Exporta el primer certificado localizado
	if (objColCertificates != null && objColCertificates.Count > 0)
		WriteFile(objColCertificates[0].Export(X509ContentType.Cert), strFileName);
	// Cierra el almacén
	objStore.Close();
}

/// <summary>
///		Escribe un array de bytes en un archivo
/// </summary>
private static void WriteFile(byte[] arrBytData, string strFileName)
{ 
	using (System.IO.FileStream fsFile = new System.IO.FileStream(strFileName, System.IO.FileMode.Create, System.IO.FileAccess.Write))
	{ 
		// Escribe el array de bytes
		fsFile.Write(arrBytData, 0, arrBytData.Length);
		// Cierra el archivo
		fsFile.Close();
	}
}
</pre>
```

## Borrado de certificados

Para borrar un certificado se deben seguir los siguientes pasos:

* Abrir el almacén con acceso `ReadWrite`
* Localizar el certificado/s a borrar
* Eliminar el certificado/s del almacén
* Cerrar el almacén
	
La clase `X509Store` contiene dos métodos diferentes para eliminar certificados:

* `Remove`: borra un certificado
* `RemoveRange`: borra una colección de certificados
	
Dado que una búsqueda por nombre en un almacén puede devolver varios certificados podríamos crear un método para borrar 
todos los certificados con un nombre determinado utilizando el método `RemoveRange`:

```csharp
/// <summary>
///		Borra los certificados de un almacén
/// </summary>
public static void DeleteCertificate(string strCertificateName, X509Store objStore)
{ 
	X509Certificate2Collection objColCertificates;
	
	// Abre el almacén
	objStore.Open(OpenFlags.ReadWrite);
	// Localiza los certificados
	objColCertificates = objStore.Certificates.Find(X509FindType.FindBySubjectName, strCertificateName, true);
	// Elimina los certificados localizados
	if (objColCertificates != null && objColCertificates.Count > 0)
		objStore.RemoveRange(objColCertificates);
	// Cierra el almacén
	objStore.Close();
}
```
		
Aunque también podríamos realizar una búsqueda por nombre pero borrar únicamente el que tenga determinada firma utilizando el 
método `Remove`:

```csharp
/// <summary>
///		Borra los certificados de un almacén
/// </summary>
public static void DeleteCertificate(string strCertificateName, string strThumbPrint, X509Store objStore)
{ 
	X509Certificate2Collection objColCertificates;
	
	// Abre el almacén
	objStore.Open(OpenFlags.ReadWrite);
	// Obtiene los certificados
	objColCertificates = objStore.Certificates.Find(X509FindType.FindBySubjectName, strCertificateName, true);
	// Elimina el certificado adecuado
	if (objColCertificates != null && objColCertificates.Count > 0)
	{ 
		bool blnDeleted = false;
		
		foreach(X509Certificate2 objCertificate in objColCertificates)
			if (!blnDeleted && objCertificate.Thumbprint.Equals(strThumbPrint, StringComparison.CurrentCultureIgnoreCase))
			{ 
				// Elimina el certificado	
				objStore.Remove(objCertificate);
				// Indica que se ha borrado
				blnDeleted = true;
			}
	}
	// Cierra el almacén
	objStore.Close();
}
```