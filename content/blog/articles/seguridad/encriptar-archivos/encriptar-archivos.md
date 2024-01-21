+++
title = "Encriptar / desencriptar archivos en C#"
date = "2019-10-11"
description = "Una forma rápida de encriptar archivos en .NET a partir de la versión 2.0"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

Desde la versión 2.0 del Framework de.NET existe una forma sencilla de encriptar y desencriptar
archivos en C#.
	
Para ello simplemente tenemos que utilizar los métodos `Encrypt` y `Decrypt` de la clase
`File` del espacio de nombres `System.`**:

De hecho, los métodos son tan sencillos que lo único que tenemos que hacer es llamar a los métodos
con el nombre de archivo que deseamos encriptar o desencriptar:
	
```csharp
File.Encrypt("C:\\temp\\test.txt");
File.Decrypt("C:\\temp\\test.txt");
```

Una vez encriptado / desencriptado, el resultado queda en el mismo archivo pasado como parámetro (por supuesto, el
archivo no puede estar en uso cuando llamemos a los métodos o nos devolverá una excepción).

Para la encriptación y desencriptación del archivo se utiliza el proveedor de servicios criptográficos del
sistema operativo (CSP - Cryptographic Service Provider), por eso impone algunas restricciones:

* El sistema de archivos debe estar formateado como NFTS.
* El sistema operativo debe ser Windows NT o superior.
* Puede que no funcione con las versiones más reducidas de Windows Vista o Windows 7 como Home o Basic.