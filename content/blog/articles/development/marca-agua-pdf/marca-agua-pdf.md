+++
title = "Crear marca agua en PDF utilizando iTextSharp"
date = "2019-10-11"
description = "Cómo crear una marca de agua en un PDF utilizando iTextSharp"
thumbnail = "/images/noimage.jpg"
tags = [ "Utilidades" ]
+++

Una de las preguntas más comunes cuando se commienza a trabajar con **iTextSharp** es cómo
crear una marca de agua (watermark) en una página de un PDF.

La respuesta es bastante sencilla, simplemente debemos crear una nueva imagen y establecer
que se debe utilizar como fondo de página.

El código sería similar al siguiente:

```csharp
private void WriteWaterMark(Document objPdfDocument, string strFileImage) 
{ 
	Image objImagePdf;
	
	// Crea la imagen
	objImagePdf = Image.GetInstance(strFileImage);
	// Cambia el tamaño de la imagen
	objImagePdf.ScaleToFit(3000, 770);
	// Se indica que la imagen debe almacenarse como fondo
	objImagePdf.Alignment = iTextSharp.text.Image.UNDERLYING;
	// Coloca la imagen en una posición absoluta
	objImagePdf.SetAbsolutePosition(7, 69);
	// Imprime la imagen como fondo de página
	objPdfDocument.Add(objImagePdf);
}
```

Los pasos son muy sencillos, simplemente creamos la imagen leyendo un archivo, escalamos la imagen y cambiarlo
para establecer el fondo utilizando **Image.UNDERLYING**. Una vez hecho esto, simplemente debemos añadirlo a la 
página en la posición que deseemos.

Por supuesto, podemos dejar la imagen con su tamaño original y colocarla donde queramos, no es necesario
ajustarla a la página.

Si simplemente deseamos establecer el fondo de la página utilizando un color sólido, lo que debemos hacer es crear
un rectángulo del color deseado que ocupe toda la página y añadir este rectángulo al PDF:

```csharp
private void WriteWaterMark(Document objPdfDocument, System.Drawing.Color clrColor) 
{ 
	Rectangle objRectangle = new Rectangle(objPdfDocument.PageSize);
	
	// Asigna el color de fondo
	objRectangle.BackgroundColor = new BaseColor(clrColor);
	// Añade el rectángulo a la página
	objPdfDocument.Add(objRectangle);
}
```

Por supuesto, ambas técnicas se pueden combinar, podemos asignar primero el rectángulo con el color
del fondo de la página y posteriormente añadir la imagen de la marca de agua.