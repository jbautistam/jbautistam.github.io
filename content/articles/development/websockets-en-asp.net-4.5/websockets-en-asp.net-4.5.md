+++
title = "Soporte para WebSockets de HTML 5 en ASP.Net 4.5"
date = "2019-10-11"
description = "El nuevo ASP.Net 4.5 ofrece la posibilidad de utilizar las funciones de WebSocket de HTML 5 para comunicaciones entre cliente y servidor"
thumbnail = "/images/noimage.jpg"
tags = [ "Comunicaciones", "Programación" ]
+++

El protocolo de **WebSocket** es un protocolo estándar para comunicaciones seguras, bidireccionales
y en tiempo real entre cliente / servidor utilizando HTTP y facilita las tareas de transferencia
de datos entre diferentes ordenadores.

**WebSocket** apareció acompañando al nuevo **HTML 5** aunque puede utilizarse sobre cualquier cliente
no únicamente para los navegadores.

La nueva versión de ASP.Net (ASP.Net 4.5) junto con IIS 8 incluye la posiblidad de utilizar el 
protocolo de **WebSockets** a bajo nivel con APIs administradas que permiten leer y enviar datos
tanto en forma de cadena como binarios de forma asíncrona. Para ello se ha incluído un nuevo espacio
de nombres **System.Web.WebSockets** que trata este prococolo.

Desde el navegador (por ejemplo) se puede establecer una conexión que apunte a una URL de una aplicación
ASP.Net de la siguiente forma:

```csharp
socket = new WebSocket("ws://server.com/Aplicacion.ashx");
```

En la parte de la aplicación ASP.Net se pueden establecer puntos de entrada utilizando módulos con archivos
ashx.

La aplicación ASP.Net debe aceptar la solicitud de abrir una conexión **WebSockets** de esta forma:

```csharp
HttpContext.Current.AcceptWebSocketRequest(fncDelegado)
```

El método **AcceptWebSocketRequest** utiliza un delegado al que transferir el control una vez
que llegan los datos de **WebSocket**. Una vez establecida la comunicación entre el cliente y el servidor,
ASP.Net realizará las llamadas oporturnas al delegado. La siguiente función muestra cómo lograr un eco de lo que
se manda a través de **WebSocket**:

```csharp
public async Task MyWebSocket(AspNetWebSocketContext context)
{ 
	WebSocket socket = context.WebSocket;
	
	while (true)
	{ 
		ArraySegment<byte> buffer = new ArraySegment<byte>(new byte[1024]);
		
		// Espera que llegue un mensaje del cliente
		WebSocketReceiveResult result = await socket.ReceiveAsync(buffer, CancellationToken.None);
		// Si el socket está abierto, recoge el mensaje del cliente
		if (socket.State == WebSocketState.Open)
		{ 
			string userMessage = Encoding.UTF8.GetString(buffer.Array, 0, result.Count);
			
			// Recoge el mensaje
			userMessage = "Se envió: " + userMessage + " a " + DateTime.Now.ToLongTimeString();
			buffer = new ArraySegment<byte>(Encoding.UTF8.GetBytes(userMessage));
			// Envía un mensaje asíncrono al cliente
			await socket.SendAsync(buffer, WebSocketMessageType.Text, true, CancellationToken.None);
		}
		else //... sale del bucle de espera de mensajes
			break; 
	}
}
```

En el ejemplo se utilizan las palabras clave **await** y **asynchronous** de las operaciones
para el tratamiento asíncrono de tareas. La aplicación espera el mensaje enviado por el cliente llamando a
**await socket.ReceiveAsync**. Del mismo modo se puede enviar un mensaje al cliente utilizando
**await socket.SendAsync**.

En el navegador (o la aplicación cliente) se reciben los mensajes de **WebSocket** a través de
la función **onmessage**. Para enviar un mensaje desde el navegador, se puede llamar al método
**send** del tipo **WebSocket** de DOM como en este ejemplo:

```csharp
// Recibe una cadena desde el servidor
socket.onmessage = function(msg)
{ 
	document.getElementById("serverData").innerHTML = msg.data; 
};

// Envía una cadena desde el navegador
socket.send(document.getElementById("Texto del mensaje enviado al servidor"));
```

Fuente: [ASP.Net](http://www.asp.net/vnext/whats-new)