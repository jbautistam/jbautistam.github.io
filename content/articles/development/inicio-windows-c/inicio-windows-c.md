+++
title = "Añadir una aplicación al inicio de Windows en C#"
date = "2019-10-11"
description = "Cómo hacer que una aplicación se inicie cuando se arranque Windows en C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Sistema" ]
+++

Para que una de nuestras aplicaciones (o cualquier ejecutable) se ejecute al **inicio de Windows** lo único realmente
necesario es añadir la ruta de la aplicación a la clave `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`
del registro.

Así, podríamos utilizar una rutina como esta:

```csharp
public static void AddStartApp(string strName, string strExecutable)
{	
	RegistryKey objKey = Registry.CurrentUser.OpenSubKey("SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run", true);
	
	objKey.SetValue(strName, strExecutable);
}
```

En la rutina anterior tenemos dos parámetros:

* `strName`: es el nombre del elemento dentro de la rama con el que vamos a identificar al ejecutable. Este nombre debe ser
único, si ya existe en esa rama del **registro** lo que ocurriría es que sobreescribiríamos el valor.
* `strExecutable`: es el nombre del archivo ejecutable (incluyendo el directorio) que deseamos ejecutar al inicio de Windows.

Eliminar la aplicación del inicio es igualmente sencillo, simplemente tenemos que eliminar el valor de la clave de registro anterior:

Hay que tener en cuenta que al utilizar `Registry.CurrentUser` estamos obteniendo la rama de registro del usuario, es decir,
añadimos la aplicación al inicio de la sesión de ese usuario, si lo que deseamos es añadir la aplicación al inicio de sesión
de todos los usuarios de la máquina, debemos utilizar la misma clave pero con `Registry.LocalMachine`.

```csharp
public static void DeleteStartApp(string strName)
{	
	RegistryKey objKey = Registry.LocalMachine.OpenSubKey("SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run", true);
	
	objKey.deletevalue(strName);
}
```