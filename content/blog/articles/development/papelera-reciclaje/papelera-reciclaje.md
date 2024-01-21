+++
title = "Borrar un archivo y enviarlo  la papelera de reciclaje en C#"
date = "2019-10-11"
description = "Cómo borrar un archivo y enviarlo a la papelera de reciclaje utilizando C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Sistema" ]
+++

Las funciones de C# para borrar archivos y directorio sólo permiten eliminar los datos, no nos permiten
enviarlos a la **papelera de reciclaje**.

Para borrar el archivo enviándolo a la papelera de reciclaje, debemos utilizar las funciones de la API, en concreto
el método **SHFileOperation**.

Para ello, primero definimos las funciones y las constantes que se utilizan en la llamada a la API:

```csharp
[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto, Pack = 1)]
public struct SHFILEOPSTRUCT
{
	public IntPtr hwnd;
	[MarshalAs(UnmanagedType.U4)]
	public int wFunc;
	public string pFrom;
	public string pTo;
	public short fFlags;
	[MarshalAs(UnmanagedType.Bool)]
	public bool fAnyOperationsAborted;
	public IntPtr hNameMappings;
	public string lpszProgressTitle;
}

[DllImport("shell32.dll", CharSet = CharSet.Auto)]
public static extern int SHFileOperation(ref SHFILEOPSTRUCT FileOp);

public const int FO_DELETE = 3;
public const int FOF_ALLOWUNDO = 0x40;
public const int FOF_NOCONFIRMATION = 0x10; // No pregunta al usuario
```

Una vez definidas las funciones y las constantes, se puede borrar el archivo o directorio enviándolo a la **papelera de reciclaje**
de esta forma:

```csharp
SHFILEOPSTRUCT shf = new Win32.SHFILEOPSTRUCT();

shf.wFunc = Win32.FO_DELETE;
shf.fFlags = Win32.FOF_ALLOWUNDO;
shf.pFrom = @"c:\archivo.txt" + '\0' + '\0';
Win32.SHFileOperation(ref shf);
```

El parámetro **pFrom** puede ser tanto un archivo como un directorio.