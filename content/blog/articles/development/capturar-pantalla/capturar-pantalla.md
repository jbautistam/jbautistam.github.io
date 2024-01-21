+++
title = "Cómo capturar la pantalla utilizando C#"
date = "2019-10-11"
description = "Cómo realizar capturas de pantalla mediante código utilizando C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Sistema" ]
+++

**Capturar una imagen** utilizando C# es un procedimiento bastante sencillo, simplemente debemos
crear una imagen con el tamaño de la pantalla y copiar la imagen de la pantalla utilizando el método
`CopyFromScreen` que nos ofrece el objeto `Graphics`.

Lo primero que debemos hacer es incorporar los espacios de nombres `Drawing` y `Drawing.Imaging` para
el tratamiento de las imágenes:

```csharp
using System.Drawing;
using System.Drawing.Imaging;
```

En segundo lugar, creamos el objeto en el que vamos a dejar las imágenes con el tamaño apropiado
a nuestra resolución de pantalla:

```csharp
Bitmap objImage = new Bitmap(Screen.PrimaryScreen.Bounds.Width, Screen.PrimaryScreen.Bounds.Height);
```

Creamos un objeto `Graphics` para utilizar el método `CopyFromScreen()` para copiar la imagen
de pantalla en el bitmap que acabamos de crear:

```csharp
Graphics objGraphics = Graphics.FromImage(objBitmap);

objGraphics.CopyFromScreen(0, 0, 0, 0, objBitmap.Size);
```

El último paso sería grabar la imagen final en un archivo: 

```csharp
objBitmap.Save("c:\\screen.gif", System.Drawing.Imaging.ImageFormat.Gif);
```