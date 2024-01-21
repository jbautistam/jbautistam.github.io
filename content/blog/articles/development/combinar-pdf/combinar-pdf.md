+++
title = "Mezclar archivos PDF con C# e iTextSharp"
date = "2019-10-11"
description = "Código para combinar archivos PDF con C# utilizando la librería iTextSharp"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Utilidades" ]
+++

La librería [iTextSharp](http://sourceforge.net/projects/itextsharp), es una traducción en C# del código de 
**iText** en Java para el tratamiento de archivos **PDF**.
	
Aparte de permitirnos crear archivos **PDF** dinámicos, tiene funciones para 
[firmar PDF](Aplicaciones\iTextSharpSign) o combinar archivos.
	
Si lo que deseamos es combinar una serie de archivos PDF en primer lugar debemos incluir los espacios de nombres
de la librería:

```csharp
using iTextSharp.text;
using iTextSharp.text.pdf;
```

Y posteriormente ir leyendo uno a uno los archivos que deseemos combinar y grabando las páginas en otro archivo.

La siguiente función realiza la combinación de los archivos. Espera dos parámetros, el primero de ellos es el nombre
del archivo final, el segundo es un array con los nombres de los archivos que deseamos combinar:
	
```csharp
/// <summary>
///		Combina una serie de archivos PDF
/// </summary>
internal static bool Merge(string strFileTarget, string [] arrStrFilesSource)
{ 
	bool blnMerged = false;
	
	// Crea el PDF de salida
	try
	{ 
		using (System.IO.FileStream stmFile = new System.IO.FileStream(strFileTarget, System.IO.FileMode.Create))
		{ 
			Document objDocument = null;
			PdfWriter objWriter = null;
			
			// Recorre los archivos
			for (int intIndexFile = 0; intIndexFile < arrStrFilesSource.Length; intIndexFile++)
			{ 
				PdfReader objReader = new PdfReader(arrStrFilesSource[intIndexFile]);
				int intNumberOfPages = objReader.NumberOfPages;
				
				// La primera vez, inicializa el documento y el escritor
				if (intIndexFile == 0)
				{ 
					// Asigna el documento y el generador
					objDocument = new Document(objReader.GetPageSizeWithRotation(1));
					objWriter = PdfWriter.GetInstance(objDocument, stmFile);
					// Abre el documento
					objDocument.Open();
				}
				// Añade las páginas
				for (int intPage = 0; intPage < intNumberOfPages; intPage++)
				{ 
					int intRotation = objReader.GetPageRotation(intPage + 1);
					PdfImportedPage objPage = objWriter.GetImportedPage(objReader, intPage + 1);
					
					// Asigna el tamaño de la página
					objDocument.SetPageSize(objReader.GetPageSizeWithRotation(intPage + 1));
					// Crea una nueva página
					objDocument.NewPage();
					// Añade la página leída
					if (intRotation == 90 || intRotation == 270)
						objWriter.DirectContent.AddTemplate(objPage, 0, -1f, 1f, 0, 0, 
						            objReader.GetPageSizeWithRotation(intPage + 1).Height);
					else
						objWriter.DirectContent.AddTemplate(objPage, 1f, 0, 0, 1f, 0, 0);
				}
			}
			// Cierra el documento
			if (objDocument != null)
				objDocument.Close();
			// Cierra el stream del archivo
			stmFile.Close();
		}
		// Indica que se ha creado el documento
		blnMerged = true;
	}
	catch(Exception objException)
	{ 
		System.Diagnostics.Debug.WriteLine(objException.Message);
	}				
	// Devuelve el valor que indica si se han mezclado los archivos
	return blnMerged;
}
```