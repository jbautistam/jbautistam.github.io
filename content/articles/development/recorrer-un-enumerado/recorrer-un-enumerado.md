+++
title = "Recorrer un enumerado"
date = "2019-10-11"
description = ""
thumbnail = "/images/noimage.jpg"
tags = [ "Programación" ]
+++

Lo habitual para insertar constantes en nuestro código que contenga determinados valores es utilizar un enumerado:

```csharp
public enum ModeEnum
{ 	
	Normal,
	Simplex,
	Duplex
}
```

Recorrer los valores de este enumerado es bastante simple, sólamente hay que utilizar los métodos
**GetNames()** y **GetValues()** de la clase **Enum**.

El método **GetNames()** nos devuelve los nombres del enumerado. Así, si deseamos mostrar todos los nombres
del enumerado utilizaremos este código:

```csharp
foreach (string strValue in Enum.GetNames(typeof(ModeEnum)))
	System.Diagnostics.Debug.WriteLine("Texto: " + strValue);
```

Si por el contrario queremos mostrar las valores numéricos utilizamos el método **GetValues()**:

```csharp
foreach (int intValue in Enum.GetValues(typeof(ModeEnum)))
	System.Diagnostics.Debug.WriteLine("Valor: " + intValue.ToString());
```
