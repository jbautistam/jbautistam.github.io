+++
title = "Librería POP3 en CSharp (II)"
date = "2019-10-11"
description = "Segunda parte del tutorial de uso de la librería de POP3 en C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Comunicaciones", "Programación" ]
+++

Siguiendo con la explicación de la [librería POP3 en CSharp](/blog/articles/source-code/libreria-pop3-en-csharp/libreria-pop3-en-csharp)
hoy vamos a hablar de mensajes.
	
Como vimos hace ya unos meses, se pueden descargar mensajes de correo fácilmente utilizando el método
`FetchEMail` pero una vez que tenemos este mensaje debemos interpretarlo y tratarlo adecuadamente.
	
Para los nuevos en el protocolo POP3 debemos decir que todos los mensajes que se reciben utilizando este protocolo
son cadenas en texto plano en [formato MIME](http://es.wikipedia.org/wiki/Multipurpose_Internet_Mail_Extensions).
Este formato no se utiliza únicamente en este protocolo, también se usa en SMTP, IMAP y el omnipresente REST por citar
sólo algunos ejemplos.

Por resumir diremos que en este formato se combina toda la información del correo recibido en una cadena, es decir,
incluye cabeceras con el tipo de codificación, los datos del destinatario y el remitente, 
el asunto, el cuerpo y los adjuntos. Si tenéis curiosidad por ver un ejemplo simplemente buscad en vuestro gestor de correo
una opción de "Ver mensaje original" o similar.

Para mantener este formato la librería utiliza las clases del espacio de nombres `Bau.Libraries.LibMailProtocols.Messages`,
en concreto la clase `MimeMessage` es la encargada de almacenar las propiedades del correo del remitente (From), destinatario
o destinatarios (To, CC y BCC), asunto (Subject), cuerpo del mensaje (Body) y cuerpo del mensaje en HTML (BodyHTML) y 
adjuntos (Attachment).
	
**Nota:** puede que parezca extraño que haya dos cuerpos en el mensaje (Body y BodyHTML) pero el protocolo distingue entre 
ambos formatos.
	
Si lo que deseamos es interpretar la cadena de texto que hemos recibido como salida del método `FethEMail` podemos
utilizar la clase `MimeMessageParser` de esta forma:
	
```csharp
string strMessage = objPop3.FetchEMail(1);

MimeMessage objMimeMessage = MimeMessageParser.Parse(strMessage);
```

A partir de este momento podemos utilizar las propiedades del objeto `MimeMessage` para lo que deseemos, por ejemplo,
para mostrarlo:
	
```csharp
// Propiedades del mensaje
txtFrom.Text = GetAddress(objMessage.From);
txtTo.Text = GetAddress(objMessage.To);
txtCC.Text = GetAddress(objMessage.CC);
txtSubject.Text = objMessage.Subject;
txtDate.Text = string.Format("{0:dd-MM-yyyy HH:mm}", objMessage.DateSend);
// Cuerpo del mensaje
txtMessage.Text = objMessage.Body.Replace("\n", Environment.NewLine);
```

El método `GetAddress` simplemente crea una cadena con las direcciones de correo y no tiene interés para este artículo.

Si lo que deseamos es archivar los adjuntos en nuestro disco, podemos utilizar el método `Save` de la clase `Section` (los
adjuntos se consideran secciones del correo igual que el cuerpo) de esta forma:
	
```csharp
foreach (Section objAttachment in objMimeMessage.Attachments)
	objAttachment.Save(strFileName);
```