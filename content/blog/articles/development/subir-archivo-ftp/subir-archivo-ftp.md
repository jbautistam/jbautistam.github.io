+++
title = "Subir un archivo a un servidor FTP utilizando C#"
date = "2019-10-11"
description = "Código fuente que explica cómo subir un archivo a un servidor FTP utilizando C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Comunicaciones" ]
+++

Si deseamos subir archivos a un servidor **FTP** con C#, podemos utilizar la clase **FtpWebRequest** del
espacio de nombre **System.Net** del **.NET Framework**.

La utilización es bastante sencilla, simplemente creamos una solicitud sobre el servidor **FTP** indicando
el nombre de archivo que tendrá el servidor remoto:

```csharp
FtpWebRequest ftpRequest = (FtpWebRequest) WebRequest.Create("ftp://servidorFTP/directorio/archivo.txt");
```

Una vez creada la conexión asignamos las credenciales e indicamos que deseamos enviar un archivo y algunas propiedades básicas:

```csharp
// Asigna las credenciales
ftpRequest.Credentials = new NetworkCredential("usuario", "Password");
// Asigna las propiedades
ftpRequest.Method = WebRequestMethods.Ftp.UploadFile;
ftpRequest.UsePassive = true;
ftpRequest.UseBinary = true;
ftpRequest.KeepAlive = false;
```

Y por último leemos el archivo en un **stream** y lo escribimos en el stream de FTP (para obtener el stream de
comunicaciones utilizamos el método **GetRequestStream** del objeto **FtpWebRequest** ):

```csharp
using (FileStream stmFile = File.OpenRead(strFileNameLocal))
{ 
	// Obtiene el stream sobre la comunicación FTP
	using (Stream stmFTP = ftpRequest.GetRequestStream())
	{ 
		byte[] arrBytBuffer = new byte[cnstIntLengthBuffer];
		int intRead;
								
		// Lee y escribe el archivo en el stream de comunicaciones
		while ((intRead = stmFile.Read(arrBytBuffer, 0, cnstIntLengthBuffer)) != 0)
			stmFTP.Write(arrBytBuffer, 0, intRead);
		// Cierra el stream FTP
		stmFTP.Close();
	}
	// Cierra el stream del archivo
	stmFile.Close();
}
```

El siguiente bloque muestra el código completo en un único método:

```csharp
/// <summary>
///		Envía un archivo por FTP
/// </summary>
public void Upload(string strServer, string strUser, string strPassword, 
	   string strFileNameLocal, string strPathFTP)
{ 
	FtpWebRequest ftpRequest;
	
	// Crea el objeto de conexión del servidor FTP
	ftpRequest = (FtpWebRequest) WebRequest.Create(string.Format("ftp://{0}/{1}", strServer,
														Path.Combine(strPathFTP, Path.GetFileName(strFileNameLocal))));
	// Asigna las credenciales
	ftpRequest.Credentials = new NetworkCredential(strUser, strPassword);
	// Asigna las propiedades
	ftpRequest.Method = WebRequestMethods.Ftp.UploadFile;
	ftpRequest.UsePassive = true;
	ftpRequest.UseBinary = true;
	ftpRequest.KeepAlive = false;
	// Lee el archivo y lo envía
	using (FileStream stmFile = File.OpenRead(strFileNameLocal))
	{ 
		// Obtiene el stream sobre la comunicación FTP
		using (Stream stmFTP = ftpRequest.GetRequestStream())
		{ 
			byte[] arrBytBuffer = new byte[cnstIntLengthBuffer];
			int intRead;
										
			// Lee y escribe el archivo en el stream de comunicaciones
			while ((intRead = stmFile.Read(arrBytBuffer, 0, cnstIntLengthBuffer)) != 0)
				stmFTP.Write(arrBytBuffer, 0, intRead);
			// Cierra el stream FTP
			stmFTP.Close();
		}
		// Cierra el stream del archivo
		stmFile.Close();
	}
}
```

**Nota:** si desea información sobre cómo descargar un archivo de un servidor FTP
lea el artículo: [descargar archivo FTP](/blog/articles/development/descargar-archivo-ftp/descargar-archivo-ftp).
