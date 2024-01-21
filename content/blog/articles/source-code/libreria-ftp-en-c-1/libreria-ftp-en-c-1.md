+++
title = "Librería FTP en C#"
date = "2019-10-11"
description = "Librería para FTP, FTPs y FTPes escrita en C#"
thumbnail = "/articles/source-code/libreria-ftp-en-c-1/libreria-ftp-en-c-1.jpg"
tags = [ "Programación", "Comunicaciones" ]
+++

En las aplicaciones que utilizan [FTP](https://en.wikipedia.org/wiki/File_Transfer_Protocol) 
para intercambio de archivos, hasta ahora he utilizado un wrapper sobre la librería de FTP de.NET.

En uno de mis últimos proyectos, necesitaba acceder a **FTPs** y no pude conseguirlo a través de
las librerías básicas de.NET. En algunos casos me daban errores de conexión a través de un proxy HTTP y en
otros errores de comunicación. Existen formas de solventarlo pero me pareció más eficaz e instructivo implementarlo por mí
mismo.
	
Así que armado con los conocimientos adquiridos en la programacación de mi
[servicio de SMTP para desarrollo](/blog/articles/source-code/servidor-smtp/servidor-smtp) y la información del 
[protocolo FTP](https://www.w3.org/Protocols/rfc959/) me puse manos a la obra. El resultado
es una librería de acceso tanto a FTP como a FTPs o FTPes cuyo código fuente, como siempre, se puede encontrar
en [GitHub](https://github.com/jbautistam/LibFtpClient).

**Nota:** como base para la librería utilicé [ArxOne.FTP](https://github.com/ArxOne/FTP)
que funciona bastante bien pero añade múltiples sesiones por conexión. Esto no supone ningún problema si realmente se necesitan utilizar
varias sesiones, si no, complica bastante el código y nos obliga a utilizar bloqueos de acceso para poder trabajar con diferentes hilos.

## Conexión a un servidor FTP

Para utilizar la librería, por supuesto lo primero es conectar al servidor. El código de conexión es bastante sencillo:

```csharp
using Bau.Libraries.LibFtpClient;
using Bau.Libraries.LibFtpClient.Parameters;

//....

using (FtpClient objFtpClient = new FtpClient(FtpClient.FtpProtocol.FtpS, strServer, 
                                              intPort, new System.Net.NetworkCredential(strUser, strPassword), 
                                              new FtpClientParameters(blnPassive, FtpClientParameters(blnPassive, 
                                                                      FtpClientParameters.FtpProtection.FtpS, 
                                                                      FtpClientParameters.SslType.Default)))
{ 
	// Conecta al servidor
	objFtpClient.Connect();
	//..... aquí irían las funciones 
	// Desconecta del servidor
	objFtpClient.Disconnect();
}
```

El enumerado del modo (`FtpClient.FtpProtocol`) nos indica si deseamos conectar utilizando FTP, FTPs o FTPes. En los
parámetros de la conexión indicamos si accedemos en modo pasivo y los métodos de protección y conexión por SSL.

## Tratamiento de directorios

Una vez conectados, para obtener, crear, borrar o cambiar el directorio actual utilizamos los siguientes métodos:

```csharp
string strActualPath = objFtpClient.Commands.GetActualPath();
objFtpClient.Commands.ChangeDir("/dev");
objFtpClient.Commands.MakeDir("/dev/test");
objFtpClient.Commands.MakeDirRecursive("/dev/hola/test/pruebas/");
objFtpClient.Commands.RemoveDirectory("/dev/hola/test/pruebas/");
```

Como podemos observar, en el manejo de directorios y archivos, se utiliza la barra de Unix en lugar de la barra invertida de Windows. En
realidad, podemos utilizar cualquiera de las dos. La librería modifica el separador según la plataforma del servidor FTP.

## Tratamiento de archivos

Por supuesto, además de con directorios podemos tratar con archivos. Tenemos funciones para borrar un archivo o cambiarle el nombre:

```csharp
objFtpClient.Commands.Delete(strFileRemote);
objFtpClient.Commands.Rename(strFileRemoteOld, strFileRemoteNew);
```

Si deseamos ver el listado de archivos del directorio actual utilizamos una función como la siguiente:

```csharp
IList<FtpEntry> objColFiles = objFtpClient.Commands.List(cnstStrPathRemote)
foreach (FtpEntry objFile in objColFiles)
	Console.WriteLine("{0} - {1} - {2} - {3}", 
                      objFile.Name, objFile.Type == FtpEntry.FtpEntryType.Directory? "Directorio": "Archivo",
                      objFile.Size, objFile.Date);
```

El método `List` utiliza el comando `LIST` de FTP para obtener los archivos. En algunos servidores se puede utilizar
también el comando `MLST` que en teoría ofrece más información. Para ello podemos utilizar el método:
	
```csharp
IList<FtpEntry> objColFiles = objFtpClient.Commands.MList(cnstStrPathRemote)
```

## Subida y descarga de archivos al servidor FTP

Por supuesto, lo más importante de una librería de FTP es que nos permita subir y descargar archivos. Los métodos
para ésto son los siguientes:
	
```csharp
objFtpClient.Commands.Upload(strFileLocal, strFileRemote);
objFtpClient.Commands.Download(strFileRemote, strFileLocal);
```

## Código fuente

Como comentábamos al principio, podemos encontrar el código fuente de la 
[librería cliente de FTP](https://github.com/jbautistam/LibFtpClient) en **GitHub**.

Aparte del código de la librería (proyecto `LibFtpClient`) en la solución se incluye el proyecto de consola
`TestLibFtp` que muestra los parámetros de conexión y cómo utilizar los diferentes comandos y modos de conexión.
	
Por cierto, la librería no trata con el protocolo [sFTP](https://es.wikipedia.org/wiki/SSH_File_Transfer_Protocol) 
que aunque pueda parecer lo mismo que **FTPs**, en realidad es un protocolo
completamente diferente. Si necesitan una librería de sFTP les propongo [SSH.Net](https://sshnet.codeplex.com/).
