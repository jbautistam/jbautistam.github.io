+++
title = "Firma digital de archivos XML en C#"
date = "2019-10-11"
description = "Cómo firmar archivos XML utilizando un certificado digital y C#"
thumbnail = "/articles/development/firma-xml/firma-xml.jpg"
tags = [ "Programación", "Seguridad" ]
+++

Cuando se trabaja con archivos XML, en ocasiones nos solicitan que este archivo se firme digitalmente
utilizando una clave privada.

[XmlDSignature](http://www.w3.org/TR/xmldsig-core/) es un protocolo estándar para 
la firma de cadenas XML y define una serie de nodos que se añaden a un archivo XML para firmarlo. La
buena noticia es que.NET tiene una librería específica para el tratamiento de este tipo de archivos bajo el
espacio de nombres **System.Security.Cryptography.Xml**.

Las funciones de esta librería nos permiten añadir los nodos de la firma a los nodos de un documento.
Es decir, si tenemos un documento de este estilo:

```XML
<documento>
<hijo id="id_nodo">
	<elemento>
		Contenido del elemento
	</elemento>
</hijo>
</documento>
```	

Una vez invocados los métodos de firma, nos añadirá al final del documento, los nodos de la firma digital definidos 
en **XmlDSignature**. Por ejemplo:

```csharp
<documento>
...
<Signature xmlns="http://www.w3.org/2000/09/xmldsig#">
<SignedInfo>
...
<Reference URI="">
 <Transforms>
   <Transform Algorithm="http://www.w3.org/TR/1999/REC-xpath-19991116">
     <XPath xmlns:dsig="&dsig;">
     
     </XPath>
   </Transform>
 </Transforms>
 <DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
 <DigestValue></DigestValue>
</Reference>
</SignedInfo>
<SignatureValue></SignatureValue>
</Signature>
...
</documento>
```	

El código para firmar un archivo XML utilizando un [certificado digital](/blog/articles/seguridad/certificados-digitales/certificados-digitales)
es el siguiente:

```csharp
/// <summary>
///	Firma un archivo XML con un certificado
/// </summary>
public void SignXmlDocument(XmlDocument objXMLDocument, X509Certificate2 objCertificate, string strReferenceToSign = "")
{ 
	SignedXml objSignedXml = new SignedXml(objXMLDocument);
	
	// Añade la clave al documento SignedXml
	objSignedXml.SigningKey = (RSACryptoServiceProvider) objCertificate.PrivateKey;
	// Asigna el identificador de referencia
	if (!string.IsNullOrWhiteSpace(strReferenceToSign))
		objSignedXml.AddReference(GetReference(strReferenceToSign));
	// Añade la información de los parámetros de firma
	objSignedXml.Signature.KeyInfo = GetKeyInfoFromCertificate(objCertificate);
	// Calcula la firma
	objSignedXml.ComputeSignature();
	// Añade el elemento firmado al documento XML
	objXMLDocument.DocumentElement.AppendChild(objXMLDocument.ImportNode(objSignedXml.GetXml(), true));
}

/// <summary>
///	Obtiene los datos de la referencia
/// </summary>
private Reference GetReference(string strReferenceToSign)
{ 
	Reference objXmlReference = new Reference();
	
	// Crea una referencia a firmar
	objXmlReference.Uri = "#" + strReferenceToSign;
	// Añade una transformación a la referencia
	objXmlReference.AddTransform(new XmlDsigEnvelopedSignatureTransform());
	// Devuelve la referencia creada
	return objXmlReference;
}

/// <summary>
///	Obtiene la información de la firma asociada al certificado digital
/// </summary>
private KeyInfo GetKeyInfoFromCertificate(X509Certificate2 objCertificate)
{ 
	KeyInfo objKeyInfo = new KeyInfo();
	
	// Añade la cláusula con el certificado
	objKeyInfo.AddClause(new KeyInfoX509Data(objCertificate));
	// Devuelve la información
	return objKeyInfo;
}
```

El método **SignXmlDocument** espera tres parámetros: el documento XML, el certificado con el que se va a firmar y
la referencia a firmar.

De los tres parámetros, el único extraño es precisamente la referencia a firmar. La referencia nos permite firmar sólo
una sección del archivo XML y no todo el archivo. Así, en caso que necesitemos firmar sólo los hijos de un nodo, pasaríamos
en este parámetro el id del nodo XML que se debe firmar.

Así en el ejemplo de archivo XML que veíamos al principio:

```csharp
<pre class="language-markup">
<code class="language-markup">
<documento>
<hijo id="id_nodo">
	<elemento>
		Contenido del elemento
	</elemento>
</hijo>
</documento>
</code>
</pre>
```	

si quisiéramos firmar únicamente el nodo **hijo** le pasaríamos a la función el valor 'id_nodo' como tercer parámetro.