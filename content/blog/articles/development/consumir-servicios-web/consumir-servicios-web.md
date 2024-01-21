+++
title = "Consumir servicios Web utilizando SSL/HTTPS en C#"
date = "2019-10-11"
description = "Descripción de la forma de consumir servicios Web utilizando protocolos SSL/HTTPS en C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Comunicaciones", "Programación" ]
+++

En ocasiones, cuando se consumen **servicios Web** utilizando **SSL / HTTPS** puede que encontremos errores de este tipo:

> Se ha cerrado la conexión subyacente: no se han podido establecer relaciones de confianza con el servidor remoto / 
> The underlying connection was closed: Could not establish trust relationship with remote server."

En estos casos, si conectamos a la URL del **servicio Web** utilizando el explorador, veremos en ocasiones un cuadro de diálogo
de confirmación sobre el certificado o similar, por ejemplo, un cuadro de diálogo que nos informa que el certificado está 
caducado y si deseamos seguir confiando en él.

Existe una forma de automatizar la respuesta a estos cuadros de diálogo y así evitar que nuestra llamada al **servicio Web**
lance una excepción. Consiste en implementar la función `CheckValidationResult` del interface `ICertificatePolicy`.
Este interface decide si se puede confiar o no en un certificado dependiendo del error. 

Por ejemplo, podríamos implementar el interface de la forma más simple posible indicando que vamos a confiar en todos los
certificados independientemente del error:

```csharp
using System.Net;
using System.Net.Security;
using System.Security.Cryptography.X509Certificates;

....

public class CertificatePolicy: System.Net.ICertificatePolicy
{
    public CertificatePolicy() 
    {
    }
    
    public bool CheckValidationResult(ServicePoint sp, X509Certificate cert, WebRequest req, int problem)
    { 
    	return true;
    }
}
```

Una vez implementado el interface, antes de realizar cualquier solicitud a la red (con `WebRequest` o similar) 
debemos indicar que todas los errores de certificado se deben validar utilizando nuestra clase. Es muy sencillo, 
simplemente cambiamos la propiedad `CertificatePolicy` del objeto `ServicePointManager`:

```csharp
ServicePointManager.CertificatePolicy = new TrustAllCertificatePolicy();
```

En las últimas versiones de.NET, aunque el sistema sigue funcionando, esta forma de hacerlo está marcada como
**obsoleta**. Para solucionarlo, en las últimas versiones, en lugar de crear una clase implementando una interfaz,
asignaremos una función al método `ServerCertificateValidationCallback`
podemos seguir las instrucciones para cambiar este método por un delegado:

```csharp
using System.Net;
using System.Net.Security;
using System.Security.Cryptography.X509Certificates;

....

ServicePointManager.ServerCertificateValidationCallback =
	delegate(object objSender, X509Certificate objCertificate, 
			 X509Chain objChain, SslPolicyErrors objSslPolicyErrors)
		{ 
			return true; //... descarta todos los errores
		};
```

De esta forma podemos saltarnos todos los errores en la lectura de los certificados. Si sólo deseamos que se considere el certificado
	como válido en el caso que no haya errores utilizaríamos este otro código:

```csharp
using System.Net;
using System.Net.Security;
using System.Security.Cryptography.X509Certificates;

....

ServicePointManager.ServerCertificateValidationCallback =
	delegate(object objSender, X509Certificate objCertificate,
			 X509Chain objChain, SslPolicyErrors objSslPolicyErrors)
		{ 
			return sslPolicyErrors == SslPolicyErrors.None; 
		};
```

Por supuesto, la utilización del último parámetro nos ofrece muchas posibilidades aunque esto lo dejaremos para
próximos artículos.