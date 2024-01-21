+++
title = "Servidor SMTP para desarrollo (C#)"
date = "2019-10-11"
description = "Código fuente de un servidor SMTP para desarrollo escrito en C#"
thumbnail = "/articles/source-code/servidor-smtp/servidor-smtp.jpg"
tags = [ "Programación", "Comunicaciones" ]
+++

Este fin de semana tenía la intención de recuperar el código de una librería antigua para generación de informes y aprovechar
el tratamiento de plantillas de [NSharpDoc]/articles/source-code/documentacion-codigo-csharp-plantillas/documentacion-codigo-csharp-plantillas)
para hacerla algo más versátil pero el primer correo electrónico de la mañana desbarató mis planes y me llevó a otro desarrollo completamente diferente.

El correo en sí era sobre una aplicación que ofrecía servidores SMTP para desarrollo, los conocidos como
`FakeSMTP`. Un `FakeSMTP` no es más que un servidor de SMTP que no envía los correos recibidos a las direcciones de correo finales. 

¿Y para qué querríamos algo así? Muy sencillo: para pruebas en desarrollo aunque, como veremos al final del artículo,
puede tener otros usos.

En desarrollo, es normal tener que enviar correos electrónicos. El problema es que mientras hacemos pruebas no queremos que estos correos
lleguen a usuarios finales. Una solución es introducir siempre direcciones de correo falsas pero irremediablemente nos encontramos
a alguien que haciendo pruebas introduce una dirección real y acabamos enviando un correo totalmente ilógico a un cliente con las
molestias que nos podemos imaginar. Otra solución puede ser trucar el código de modo que si estamos en entornos de desarrollo
o de preproducción envíen el correo a una dirección fija pero eso nos obliga a escribir código para comprobar el entorno y
ésto no siempre funciona.

La solución más idónea es utilizar un servidor de SMTP que almacene los correos de forma que podamos ver el resultado pero que no
los envíe a los destinatarios finales. Esto es un #b FakeSMTP#.

Como decía al principio, a eso dediqué mi fin de semana. Por supuesto, no es tan versátil como la solución comercial que me envió
la publicidad pero demuestra lo sencillo que es crear un servidor de SMTP enC#.

## Código fuente del servidor SMTP

Para los que no puedan esperar, las fuentes de este servidor SMTP se encuentran en 
[GitHub](https://github.com/jbautistam/DevelopSmtpServer). Este artículo explica
su funcionamiento.

## Introducción al protocolo SMTP

Cualquier servidor SMTP debe cumplir con el protocolo definido en la 
[RFC 821](https://tools.ietf.org/html/rfc821) de agosto de 1.982, todo un veterano. El protocolo
ha tenido algunos añadidos a lo largo del tiempo para tratar objetos MIME o añadir conceptos de seguridad (extrañamente, 
el protocolo inicial ni siquiera define la forma de enviar la contraseña del usuario), pero si queremos un servidor
sencillo como es este caso, nos basta con cumplir esta RFC.

En el protocolo se identifican los mensajes que recibe el servidor y las respuestas de éste. Todo el protocolo se basa en el envío
y la recepción de cadenas de texto en ASCII. Por tanto, cualquier adjunto se tiene que enviar utilizando MIME y cualquier 
carácter que se salga del ASCII de 7bits debe ir codificado. 

Sin embargo, para el proyecto que tenemos entre manos, no necesitamos interpretar esta codificación dado que 
es el cliente quien debe encargarse de enviar y traducir correctamente el contenido del correo.

Entre las cadenas de texto que se intercambian el cliente y el servidor hay siete comandos obligatorios. El cliente envía estos
comandos al comienzo de cada línea de texto seguido de una serie de parámetros. El servidor envía un número como respuesta
seguido de información adicional que normalmente no se procesa. 

Los comandos que debemos implementar son:

* **EHLO (o HELO):** es el comando que envía el cliente para iniciar la sesión. No tiene porqué llevar parámetros pero normalmente
nos encontramos con el nombre de la máquina que se está conectando con el servidor. El servidor responde a este mensaje con
una cadena de texto que comienza por **250**. Normalmente la respuesta suele ser '250 OK'.
* **MAIL FROM:** con este comando, el cliente indica el correo del remitente del mensaje. El argumento de este mensaje es precisamente
la dirección de correo origen y en algunos casos el nombre de la persona asociada al correo. La respuesta del servidor, como
en el caso anterior debe ser 250 OK.
* **RCPT TO:** este comando es similar al anterior pero identifica el destinatario del correo. Se puede enviar más de un destinatario
con varios comandos consecutivos. La respuesta a cada una de las líneas, como es ya habitual será 250 OK.
* **DATA:** indica el inicio del cuerpo del mensaje. El servidor debe responder con un valor 354. Normalmente se envía algo así
como '354 send the mail data, end with.'. A partir de ese momento el cliente envía líneas de texto con el contenido del mensaje
correctamente codificado y en formato MIME si es necesario. Para indicar el final del correo el cliente envía una línea vacía seguida
de una línea con únicamente un punto. A esta última línea el servidor debe responder con un mensaje '250 OK'.
* **QUIT:** indica el fin de sesión. A este comando el servidor responde con un mensaje 221 y puede cerrar la sesión.
* **NOOP:** este es un comando que indica que no se debe hacer nada. El servidor, de cualquier forma responde con un mensaje
'250 OK'. Este mensaje se suele utilizar para mantener la conexión abierta.
* **RSET:** comando que indica que se deseche el correo que se estaba enviando, los emisores y los destinatarios. La respuesta
debe ser '250 OK'.

Existen algunos comandos adicionales en el protocolo pero el mínimo que debe reconocer nuestro servidor son los siete anteriores.

## Creación del servidor SMTP

Como podemos anticipar por la explicación del protocolo, el servidor es bastante sencillo de escribir, nuestro mayor problema es crear
una conexión TCP al puerto adecuado y esperar los clientes de forma asíncrona para no bloquear el proceso.

En mi código, he creado una librería que implementa el protocolo SMTP y graba los correos recibidos en un directorio. Esta librería
la podéis encontrar en [GitHub](https://github.com/jbautistam/DevelopSmtpServer). Dentro de la 
solución, el proyecto en el que se programa el servidor SMTP es `LibSmtpServer`.

Dentro de esta librería, la clase `SmtpServer` es la encargada de la configuración del servidor, preparar las conexiones y recibir y
enviar eventos. Por supuesto es una clase muy sencilla, lo fundamental (para no perdernos) sería lo siguiente:
	
```csharp
public SmtpServer(string strPathBase, string strIP = "127.0.0.1", int intPort = 25)
{ 
	PathBase = System.IO.Path.Combine(strPathBase, "EMail");
	IP = strIP;
	Port = intPort;
}

public void Connect()
{ 
	// Cierra el listener
	Disconnect();
	// Crea de nuevo el listener
	Listener = new Receiver.SmtpListener(this);
	Listener.Connect();
	// Lanza el evento
	RaiseEventLog("Conexión", "Conectado");
}

public void Disconnect()
{ 
	if (Listener != null)
	{ 
		// Desconecta el listener
		Listener.Disconnect();
		// Log
		RaiseEventLog("Desconesión", "Desconectado");
	}
}

private Receiver.SmtpListener Listener { get; set; }
```
	
Lo más importante es que recogemos el directorio donde vamos a dejar los mensajes, la dirección IP y el puerto en el que vamos a escuchar.

El método `Connect` conecta el objeto que implementa el protocolo utilizando TCP en la dirección IP definida y el método `Disconnect`
desconecta y cierra el servicio.

Por supuesto, el grueso del trabajo lo hace precisamente la clase `SmtpListener`. En primer lugar, tenemos un método de conexión y otro
de desconexión:
	
```csharp
internal void Connect()
{ 
	if (!IsConnected)
	{ 
		// Indica que está conectado
		IsConnected = true;
		// Arranca el servicio de escucha
		StartListener();
	}
}

internal void Disconnect()
{ 
	StopListener();
	IsConnected = false;
}
```

El método `StartListener` utiliza un objeto [TcpListener](https://msdn.microsoft.com/es-es/library/system.net.sockets.tcplistener(v=vs.110).aspx)
de .NET. La función de este objeto básicamente es conectarse a un puerto y esperar a que se conecten los clientes de forma asíncrona:
	
```csharp
private async void StartListener() 
{ 
	// Si no existe el objeto de escucha, lo crea
	if (Listener == null)
		Listener = new TcpListener(new System.Net.IPEndPoint(System.Net.IPAddress.Parse(Server.IP), Server.Port));
	// Inicializa el listener
	Listener.Start();
	// Mientras continúe conectado
	while (IsConnected)
	{ 
		try
	    { 
		    TcpClient objTcpclient = await Listener.AcceptTcpClientAsync().ConfigureAwait(false);                
	
		        // Trata el cliente
		        HandleClient(objTcpclient);
	    }
	  	catch (Exception objException)
	  	{ 
	    	System.Diagnostics.Debug.WriteLine("Excepción: " + objException.Message);
	    }
	}
}

```

Como vemos en el método anterior, cada vez que un cliente se conecte a nuestro servidor, se pasará al método `HandleClient`
que es el que implementa el protocolo. Veamos una versión reducida del método (sin log ni verificaciones finales):
	
```csharp
private void HandleClient(TcpClient objTcpClient)
{ 
	using (NetworkStream stmNetwork = objTcpClient.GetStream())
	{ 
		using (System.IO.StreamReader stmReader = new System.IO.StreamReader(stmNetwork))
		{ 
			bool blnIsEnd = false, blnIsData = false;
			string strPreviousLine = "";
			MessageWriter objWriter = new MessageWriter();
			
			// Crea el generador de archivo
			objWriter.Create(Server.PathEMailsToSend);
			// Envía la línea de información del servidor
			Write(stmNetwork, "220 BauSMTPServer");
			// Recoge los datos
			while (!blnIsEnd)
			{ 
				string strLine = stmReader.ReadLine(); 
				
				switch (GetLineType(strLine, blnIsData))
				{ 
					case LineType.Hello:
							WriteOk(stmNetwork);
						break;
					case LineType.Quit:
							blnIsEnd = true;
							Write(stmNetwork, "221 DevelopSMTP Service closing transmission channel");
						break;
					case LineType.StartData:
							blnIsData = true;
							Write(stmNetwork, "354 send the mail data, end with.");
						break;
					case LineType.EndData:
							blnIsData = false;
							WriteOk(stmNetwork);
						break;
					case LineType.Noop:
							WriteOk(stmNetwork);
						break;
					case LineType.MailFrom:
							objWriter.WriteFrom(GetEMail(cnstStrEMailFrom, strLine));
							WriteOk(stmNetwork);
						break;
					case LineType.MailTo:
							objWriter.WriteTo(GetEMail(cnstStrEMailTo, strLine));
							WriteOk(stmNetwork);
						break;
					case LineType.Reset:
							blnReset = true;
							WriteOk(stmNetwork);
						break;
					case LineType.Unknown:
							if (blnIsData)
							{ 
								// Si la línea es un punto y la línea anterior es un salto de línea, termina la transmisión de datos
								if (strLine == "." && string.IsNullOrEmpty(strPreviousLine))  
								{ 
									// Indica que ya no hay más datos
									blnIsData = false;
									//... y envía un OK al cliente
									WriteOk(stmNetwork);
								}
								else // Añade la línea al mensaje
									objWriter.Write(strLine);
								// Guarda la línea actual para la comparación con la siguiente
								strPreviousLine = strLine;
							}
							else //... cualquier mensaje que no entienda, devuelve un OK
								WriteOk(stmNetwork);
						break;
					default: //... esto no debería pasar... siempre va a entrar por el LineType.Unknown
							WriteOk(stmNetwork);
						break;
					}
				}
			// Cierra el generador de mensajes
			objWriter.Dispose();
	  	}
	}
}
```

En este método se abre un stream de lectura sobre la conexión TCP del cliente y se van leyendo líneas de texto. Por cada una de las líneas
de texto, se llama a la función `GetLineType` que coge la cabecera del comando y la transforma en un enumerado. Dependiendo de este
enumerado se responde al cliente escribiendo sobre el stream que tenemos abierto.

La parte más 'complicada' es la que va recibiendo el contenido del correo. Dado que el protocolo nos indica que los datos 
terminan cuando recibimos un punto después de una línea vacía, nos vamos guardando la línea anterior y comparando la que hemos
recibido en el bucle y la anterior con una línea vacía y un punto respectivamente.

**Nota:** Para los que se pregunten qué ocurre cuando dentro del correo se envía una línea en blanco seguida 
por una línea con únicamente un punto (yo lo haría), este tipo de mensajes se codifica de forma que nunca pueda ocurrir algo así 
en el texto de un correo electrónico (normalmente utilizando 
[codificación Quoted Printable](https://es.wikipedia.org/wiki/Quoted_printable).

Por completar el ejemplo, vamos a ver el resto de métodos importantes:

```csharp
private LineType GetLineType(string strLine, bool blnIsData)
{ 
	LineType intType = LineType.Unknown;
	
	// Obtiene el tipo de línea
	if (!string.IsNullOrEmpty(strLine))
	{ 
		if (!blnIsData)
		{ 
			if (strLine.StartsWith("HELO", StringComparison.CurrentCultureIgnoreCase) ||
					strLine.StartsWith("EHLO", StringComparison.CurrentCultureIgnoreCase))
				intType = LineType.Hello;
			else if (strLine.StartsWith("DATA", StringComparison.CurrentCultureIgnoreCase))
				intType = LineType.StartData;
			else if (strLine.StartsWith("NOOP", StringComparison.CurrentCultureIgnoreCase))
				intType = LineType.Noop;
			else if (strLine.StartsWith("QUIT", StringComparison.CurrentCultureIgnoreCase))
				intType = LineType.Quit;
			else if (strLine.StartsWith("RSET", StringComparison.CurrentCultureIgnoreCase))
				intType = LineType.Reset;
			else if (strLine.StartsWith(cnstStrEMailFrom, StringComparison.CurrentCultureIgnoreCase))
				intType = LineType.MailFrom;
			else if (strLine.StartsWith(cnstStrEMailTo, StringComparison.CurrentCultureIgnoreCase))
				intType = LineType.MailTo;
		}
	}
	// Devuelve el tipo de línea
	return intType;
}

private void WriteOk(NetworkStream stmNetwork)
{ 
	Write(stmNetwork, "250 OK");
}

private void Write(NetworkStream stmNetwork, string strLine)
{ 
	System.Text.ASCIIEncoding objEncoder = new System.Text.ASCIIEncoding();
	byte[] arrBytBuffer = objEncoder.GetBytes(strLine + cnstStrLineSeparator);
	
	// Escribe la cadena codificada
	stmNetwork.Write(arrBytBuffer, 0, arrBytBuffer.Length);
	// Envía la cadena
	stmNetwork.Flush();
}
```

Como decíamos anteriormente, el método `GetLineType` interpreta el tipo de comando mientras que los métodos `WriteOk` y `Write`
escriben las respuestas del servidor.
	
Lo único que nos faltaría es el objeto `objWriter` de tipo `MessageWriter`. Este objeto realmente lo único que hace es escrbir líneas
de texto sobre un archivo. Habría podido utilizar directamente un stream de escritura sin necesidad de una clase nueva pero lo dejé 
separado en una clase por añadirle todas las funciones de tratamiento de directorios y en previsión de añadidos posteriores.

Como muestra alguno de los métodos de esta clase:

```csharp
internal void Create(string strPath)
{ 
	string strFileName = GetFileName(strPath);
	
	// Crea el directorio
	System.IO.Directory.CreateDirectory(System.IO.Path.GetDirectoryName(strFileName));
	// Abre el stream de escritura
	Writer = new System.IO.StreamWriter(strFileName, false, System.Text.ASCIIEncoding.ASCII);
	// Indica que está abierto
	IsDisposed = false;
}

private string GetFileName(string strPath)
{ 
	string strFileName = System.IO.Path.Combine(strPath, string.Format("{0:yyyy-MM-dd}", DateTime.Now));
	
	// Añade el nombre de archivo
	strFileName = System.IO.Path.Combine(strFileName, Guid.NewGuid().ToString() + ".msg");
	// Devuelve el nombre de archivo
	return strFileName;
}

internal void WriteFrom(string strEMail)
{ 
	Write("FROM: " + strEMail);
}

internal void WriteTo(string strEMail)
{ 
	Write("TO: " + strEMail);
}

internal void Write(string strMessage)
{ 
	if (Writer != null)
	    Writer.Write(strMessage + "\r\n");
}

private System.IO.StreamWriter Writer { get; set; }
```

Simple tratamiento de archivos. El archivo de texto que se genera consta de una línea que comienza por `FROM` con el remitente,
una o varias líneas comenzando por `TO` con el o los destinatarios y el cuerpo del mensaje tal y cómo lo hemos recibido del
servidor.

## Otros proyectos de la solución

Aparte del proyecto `LibSmtpServer` en la solución hay dos proyectos más:


* **DevelopSmtpServer:** es una aplicación que puede utilizarse como consola o como servicio de Windows y que 
utiliza `LibSmtpServer` para implementar el servidor de STMP.
* **TestSmtpserver:** es una aplicación WindowsForms para pruebas tanto de la consola como directamente de la librería. Esta aplicación
utiliza las otras dos librerías de la solución:
	* **LibHelper:** clases de ayuda y extensores.
	* **BauControls:** controles de usuario para Windows Forms.

## Ampliación del proyecto

Ya que tenemos una librería con un servidor SMTP, si queremos un servidor SMTP de desarrollo 'comercial' podemos ampliar el proyecto
para permitir ver los correos enviados desde nuestras aplicaciones desde y hacia los diferentes usuarios. Lo único que habría que
programar en este caso sería la lógica de servidor, por ejemplo en ASP.Net, para acceder a los correos enviados.

De hecho, aunque en este artículo hemos supuesto que la librería nos servía para escribir un servidor SMTP para desarrollo 
se puede ampliar para crear un servidor SMTP real, habría que implementar la lógica de reenvío de correos a un servidor 
real y el mantenimiento de usuarios.

Si lo que tenemos hasta el momento es archivos de texto con el correo que hemos recibido y las direcciones de origen y destino, nada
nos impide reenviar este texto utilizando SMTP a un servidor real. Esto nos permite por ejemplo liberar el servidor real o 
preprocesar los correos para evitar los filtros de spam lanzando sólo correos cada cierto tiempo o paralelizar el envío de 
correos entre diferentes hilos o servidores STMP.

Quién sabe... quizá en un futuro.