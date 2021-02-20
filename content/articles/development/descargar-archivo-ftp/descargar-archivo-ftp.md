+++
title = "Descargar archivo FTP"
date = "2019-10-11"
description = "Cómo descargar archivos de un servidor FTP utilizando C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Comunicaciones" ]
+++

Si deseamos descargar archivos de un servidor **FTP** con C#, podemos utilizar la clase **FtpWebRequest** del
espacio de nombre **System.Net** del **.NET Framework**.

La utilización es bastante sencilla, simplemente creamos una solicitud sobre el servidor **FTP** indicando
el nombre de archivo que deseamos descargar en el servidor remoto:

```csharp
FtpWebRequest ftpRequest = (FtpWebRequest) WebRequest.Create("ftp://servidorFTP/directorio/archivo.txt");
```

Una vez creada la conexión asignamos las credenciales e indicamos que deseamos descargar un archivo y 
algunas propiedades básicas:

```csharp
// Asigna las credenciales
ftpRequest.Credentials = new NetworkCredential("usuario", "Password");
// Asigna las propiedades
ftpRequest.Method = WebRequestMethods.Ftp.DownloadFile;
ftpRequest.UsePassive = true;
ftpRequest.UseBinary = true;
ftpRequest.KeepAlive = false;
```

Y por último leemos el archivo en un **stream** sobre el servidor FTP (utilizando el método **GetResponse()** del
objeto **FtpWebRequest** ) y lo grabamos sobre un archivo local:

```csharp
// Descarga el archivo y lo graba
using (FileStream stmFile = File.OpenWrite(strFileNameLocal))
{ 
	// Obtiene el stream sobre la comunicación FTP
	using (Stream responseStream = ((FtpWebResponse) ftpRequest.GetResponse()).GetResponseStream())
	{ 
		byte[] arrBytBuffer = new byte[cnstIntLengthBuffer];
	  	int intRead;
	
		// Lee los datos del stream y los graba en el archivo
		while ((intRead = responseStream.Read(arrBytBuffer, 0, cnstIntLengthBuffer)) != 0)
			stmFile.Write(arrBytBuffer, 0, intRead);
		// Cierra el stream FTP	
		responseStream.Close();										
	}
	// Cierra el archivo de salida
	stmFile.Flush();
	stmFile.Close();
}
```

El siguiente bloque muestra el código completo en un único método:

```csharp
/// <summary>
///	Descarga de un archivo por FTP
/// </summary>
public void Download(string strServer, string strUser, string strPassword, 
			     string strFileNameFTP, string strFileNameLocal)
{ 
	FtpWebRequest ftpRequest;
	
	// Crea el objeto de conexión del servidor FTP
	ftpRequest = (FtpWebRequest) WebRequest.Create(string.Format("ftp://{0}/{1}", strServer, 
																 strFileNameFTP));
	// Asigna las credenciales
	ftpRequest.Credentials = new NetworkCredential(strUser, strPassword);
	// Asigna las propiedades
	ftpRequest.Method = WebRequestMethods.Ftp.DownloadFile;
	ftpRequest.UsePassive = true;
	ftpRequest.UseBinary = true;
	ftpRequest.KeepAlive = false;
	// Descarga el archivo y lo graba
	using (FileStream stmFile = File.OpenWrite(strFileNameLocal))
	{ 
		// Obtiene el stream sobre la comunicación FTP
		using (Stream responseStream = ((FtpWebResponse) ftpRequest.GetResponse()).GetResponseStream())
		{ 
			byte[] arrBytBuffer = new byte[cnstIntLengthBuffer];
			int intRead;
			
				// Lee los datos del stream y los graba en el archivo
				while ((intRead = responseStream.Read(arrBytBuffer, 0, cnstIntLengthBuffer)) != 0)
					stmFile.Write(arrBytBuffer, 0, intRead);
				// Cierra el stream FTP	
				responseStream.Close();										
		}
		// Cierra el archivo de salida
		stmFile.Flush();
		stmFile.Close();
	}
}
```

**Nota:** si desea información sobre cómo subir un archivo a un servidor FTP
lea el artículo: [subir archivo por FTP](Artículos\Programación\Subir archivo FTP).