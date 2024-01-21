+++
title = "Convertir cadena hexadecimal a decimal en C# (y viceversa)"
date = "2019-10-11"
description = "Forma de convertir una cadena en hexadecimal a un entero y viceversa en C#"
thumbnail = "convertir-hexadecimal.jpg"
tags = [ "Programación" ]
+++

Una de las cosas que siempre olvido y acabo buscando por Internet.

¿Cómo se convierte una cadena en hexadecimal a un número entero en C#?

No podía ser más simple, únicamente tenemos que utilizar el método **ToInt32** (o **ToInt34** o similar) de la clase estática
**Convert** indicándole que utilice base hexadecimal (16), es decir:
	
```csharp
Convert.ToInt64(strValue, 16);
```

Donde `strValue` es nuestra cadena en hexadecimal (por ejemplo FE05A4). No hace falta ponerle los caracteres 0x por delante.

Para pasar de un entero a una cadena en hexadecimal, por el contrario, utilizamos el método `Format` de la clase `String`:

```csharp
string.format("{0:x}", decValue);
```

Muy sencillo, supongo que por eso nunca lo recuerdo.