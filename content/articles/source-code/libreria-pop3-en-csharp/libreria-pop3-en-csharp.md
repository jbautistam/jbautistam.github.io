+++
title = "Librería POP3 en C# (I)"
date = "2019-10-11"
description = "Código fuente y explicación de una librería de implementación del protocolo POP3 en C#. Parte I"
thumbnail = "/images/noimage.jpg"
tags = [ "Comunicaciones", "Programación" ]
+++

En uno de mis últimos proyectos tenía que recoger los correos recibidos en una cuenta de correo y procesarlos para guardarlos en 
una base de datos.
	
Aunque el **Framework.NET** ofrece librerías para enviar correos (protocolo **SMTP** ) no incorpora ninguna librería para
recibir corrreos electrónicos (protocolo **POP3** ), así que tuve que programar una solución prácticamente desde cero.
	
Estuve buscando por Internet implementaciones del **protocolo POP3** y aunque la mayoría hacen su trabajo correctamente, la mayor
parte son excesivamente complejas o fallan al recibir correos en español o al codificar distintos tipos de mensajes o al utilizar
SSL para conectarse a algunos servidores de correo (como cuando deseamos recibir correos de **GMail** ).
	
Al final programé una librería completamente nueva que implementa el **protocolo POP3** de la forma más sencilla posible.

El **código fuente** de esta librería se puede descargar desde [GitHub](https://github.com/jbautistam/Pop3).

## Descripción del desarrollo

La solución se divide en dos librerías:


* `LibEncoder`: contiene las clases de codificación y decodificación de cadenas o archivos en los formatos Bit7, Bit8, Quoted Printable y
Base64.
* `LibEMailProtocols`: contiene las clases relacionadas con el envío y recepción de correos.
	
`LibEncoder` es una librería bastante básica: a partir de una clase inicial (`Encoder`) podemos llamar a las funciones que realizan
la codificación / decodificación de las cadenas / archivos en el formato deseado. Por ahora no vamos a entretenernos en ella.
	
La librería `LibEMailProtocols` es la realmente interesante en este caso dado que es la que define los mensajes y protocolos tanto de
envío (**SMTP**) como de recepción (**POP3**) de correos electrónicos.
	
La solución se divide en cuatro directorios y espacios de nombres:

* `Helper`: con clases de ayuda para la utilización de la librería de envío de correos (**SMTP**).
* `Messages`: con clases para definición de los mensajes de correo (direcciones, adjuntos, cabeceras y demás) comunes tanto para
el envío como para la recepción de mensajes.
* `POP3`: con las clases que implementan el protocolo **POP3**.
* `SMTP`: con las clases que implementan el protocolo **SMTP**.
* `Tools`: con las clases estáticas que implementan los métodos de conexión a Internet y criptografía necesarios para implementar
los protocolos.

### El espacio de nombres POP3

En el espacio de nombres POP3 se encuentran las clases relacionadas más íntimamente con el protocolo. Se definen tres clases:

* `Pop3Client`: clase que implementa el protocolo POP3.
* `Pop3ClientException`: excepción lanzada por la clase anterior.
* `Pop3EventArgs`: argumentos de los eventos lanzados por la clase `Pop3Client`.
	
La clase `Pop3Client` contiene únicamente ocho métodos públicos para que sea más fácil de utilizar:

* `Connect`: conecta al servidor definido en el constructor de la clase.
* `Disconnect`: desconecta del servidor.
* `GetMailBoxDetails`: obtiene el número de correos pendientes de descargar y el tamaño de estos.
* `GetMailSize`: obtiene el tamaño de un correo determinado entre los pendientes de descargar.
* `Delete`: marca un mensaje como pendiente de borrar (se borrará cuando se desconecte del servidor.
* `Undelete`: recupera los mensajes marcados con el método anterior (siempre y cuando no se haya desconectado
aún del servidor).
* `FetchEMail`: # descarga un mensaje del servidor.
* `FetchEMailHeader`: # recupera las primeras líneas de un mensaje del servidor.
	
De esta forma el trabajo con la clase es bastante sencillo. Si deseamos descargar los correos del servidor, basta con construir la clase indicando los
parámetros de conexión al servidor, conectar, obtener el número de mensajes pendientes e ir recorriendo y descargando antes de desconectar del servidor:
	
```csharp
Pop3Client objPop3 = new Pop3Client("pop.gmail.com", 995, "user@gmail.com", 
									"password", true);
int intNumber, intSize;

	// Conecta al servidor
	objPop3.Connect();
	// Obtiene el número de correos
	objPop3.GetMailBoxDetails(out intNumber, out intSize);
	// Descarga los correos
	for (int intIndex = 0; intIndex < intNumber; intIndex++)
	{ 
		// Descarga y muestra el correo en la consola
		Console.WriteLine(objPop3.FetchEMail(intIndex + 1));
	  	// Borra el correo recibido del servidor
	  	objPop3.Delete(intIndex + 1);
	}
	// Desconecta del servidor
	objPop3.Disconnect();
```
	
En este caso nos estamos conectando al servidor de correo de **GMail** en el puerto 995 utilizando una conexión **SSL**.

Lo que obtenemos del método `FetchEMail` es una cadena con el contenido del mensaje sin ningún tratamiento. Esto resulta interesante en algunos casos, por
ejemplo en depuración, para ver el código recibido y las cabeceras. Lo normal sin embargo, es que deseemos interpretar el mensaje de forma que separemos el 
título del cuerpo y los archivos adjuntos. Para esto utilizamos las clases del espacio de nombres `Messages`.
