+++
title = "Cómo convertir SharpDevelop en una aplicación Portable"
date = "2019-10-11"
description = "Tutorial sobre cómo convertir el IDE de SharpDevelop en una aplicación portátil que podemos instalar en cualquier parte e incluso en una llave USB"
thumbnail = "/articles/development/sharpdevelop-portable/sharpdevelop-portable.jpg"
tags = [ "Utilidades" ]
+++

En ocasiones, resulta muy útil poder tener instalado en una máquina, como por ejemplo un servidor
de producción, un IDE para hacer pequeñas pruebas o depuración paso a paso, sin embargo, por 
restricciones de seguridad es posible que no podamos instalarlo sin más. En otros casos lo que 
queremos es llevar un IDE de programación con nosotros en una llave USB para utilizarla en cualquier ordenador.

Si utilizamos normalmente tecnologías.NET podemos utilizar el IDE de código abierto 
[SharpDevelop](http://www.icsharpcode.net/OpenSource/SD/) configurándolo para
que se ejecute como una **aplicación portable**.

Para conseguir que **SharpDevelop** se ejecute como **aplicación portable** simplemente debemos seguir estos
pasos:

1. Obviamente descargar **SharpDevelop** e instalarlo en nuestra máquina.
2. Copiar el directorio de **SharpDevelop** completo en la memoria USB o en la máquina en la
que lo deseemos instalar como aplicación portable.
	
A partir de aquí, ya necesitamos algo de configuración. Para ello, abrimos el directorio *bin* dentro
del directorio de la aplicación y buscar el archivo *SharpDevelop.exe.config*. En la sección
*appSettings* se muestran las opciones básicas:

```XML
<appSettings>
<!-- Use this configuration setting to store settings in a directory relative to the location
of SharpDevelop.exe instead of the user profile directory. -->
<add key="settingsPath" value="..\Settings" />

<!-- Use this setting to specify a different path for the code completion cache.
The cache contains information about referenced assemblies to speed up loading
the information on future SharpDevelop starts. -->
<add key="domPersistencePath" value="..\DomCache" />

<!-- Use this setting to disable the code completion cache. Code completion will still be
available, but take longer to load and use more RAM. -->
<add key="domPersistencePath" value="none" />
</appSettings>
```

En la configuración debemos quitar el comentario de las claves `settingsPath` y `domPersistencePath`:

```XML
<add key="settingsPath" value="..\Settings" />
<add key="domPersistencePath" value="..\DomCache" />
```

A partir de ese momento ya podemos utilizar el IDE desde nuestra USB sin necesidad de instalarlo en la máquina
cliente.
