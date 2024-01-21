+++
title = "Introducción a los certificados digitales"
date = "2019-10-11"
description = "Introducción a los certificados digitales"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

Uno de los problemas de la comunicación mediante procesos electrónicos es la imposibilidad de verificar que el emisor 
de la comunicación es realmente quien dice ser. En el Mundo Real™  para verificar la identidad de una persona u 
organización se utilizan documentos como el pasaporte, el DNI, el CIF o similares acompañados de una firma o una 
fotografía que identifica a la persona.
	
Para imitar este proceso utilizando métodos digitales se usan los certificados digitales (también conocidos 
como certificados de clave pública).
	
Los certificados digitales se basan en lo que se conoce como métodos de criptografía asimétrica en contraposición 
con los métodos de criptografía simétrica que se vienen utilizando desde el principio de los tiempos. 
	
Los métodos de criptografía simétricos se basan en la utilización de una clave que se utiliza tanto para 
encriptar como para desencriptar el mensaje. El problema es que para que el destinatario pueda desencriptar 
el mensaje previamente ha tenido que ponerse en contacto con el remitente para compartir la clave.
	
Los métodos de criptografía asimétricas vienen a resolver este problema utilizando dos claves diferentes: una 
conocida como clave pública y otra llamada clave privada, de forma que los mensajes se pueden encriptar 
utilizando la clave pública de un usuario y desencriptarlos utilizando su clave privada.
	
Esto evita que haya que compartir inicialmente una clave como se precisaba en los algoritmos simétricos aunque 
sigue siendo necesario que antes de iniciar la comunicación el remitente obtenga una copia de la clave pública 
del destinatario.
	
Al enviar un mensaje el remitente utiliza la clave pública del destinatario para encriptar el mensaje. El 
destinatario, una vez recibe el mensaje, lo único que debe hacer es utilizar su clave privada (que como su 
propio nombre indica, es secreta) para desencriptarlo.
	
La firma digital utiliza este método criptográfico para asociar la identidad de una persona o de un equipo al mensaje. 
Además asegura la integridad del mensaje, es decir, evita las modificaciones del mensaje en la transmisión entre 
emisor y receptor.
	
Para almacenar estas claves públicas y privadas que nos permiten firmar digitalmente un documento se utilizan los certificados
digitales que son documentos electrónicos (es decir, archivos) que utilizan una firma digital para asociar una clave 
pública con una identidad bien sea una persona o una organización.
	
Los certificados los emite una autoridad certificadora (CA) que actúa del mismo modo que un gobierno cuando emite un pasaporte, 
es decir, como garante de la identidad de la persona u organismo al que pertenece la clave pública. 
	
De cualquier forma, un certificado lo puede firmar tanto una autoridad certificadora, como un usuario o incluso otros 
usuarios. Los certificados firmados por los propios usuarios se denominan certificados autofirmados o certificados raíz.
	
Lo normal es que un individuo para probar su identidad confíe en una autoridad certificadora para firmar su certificado. 
Posteriormente este certificado se enviará a otro individuo para que pueda comprobar su validez contra la autoridad 
certificadora. Por supuesto, ambas partes deben confiar en la autoridad certificadora, para ello la propia entidad 
certificadora debe probar su identidad mediante otro certificado firmado por una autoridad certificadora de rango 
superior. 
	
De esto se deduce que debe existir una autoridad certificadora que no tenga una autoridad superior. Esta CA firma su
propio certificado. Este es el caso que indicábamos anteriormente en que un usuario firma su propio certificado, es 
decir, crea un certificado autofirmado o raíz.
	
Los certificados raíz se utilizan por ejemplo en la banca online accesible desde un navegador. Si tenemos una aplicación 
para dar servicio de banca online, el usuario se conecta con el sitio para ver sus datos y 
realizar transacciones. Este tipo de aplicaciones deben ser seguras, debemos impedir que el usuario acabe entrando 
en un sitio falso que robe sus credenciales y posteriormente su dinero. Para ello las conexiones a este tipo de 
servidores Web se realizan sobre HTTPS (un protocolo HTTP seguro que utiliza el protocolo SSL para identificar 
al servidor).
	
En este caso, el servidor Web permite acceder a un certificado que prueba su identidad firmado por una entidad 
certificadora reconocida. Los exploradores pueden entonces verificar la identidad del servidor bien si la entidad 
es conocida por el navegador o bien a través de una autoridad a nivel superior. Para ello, cuando el explorador se 
conecta con el servidor, recupera el certificado y lo comprueba con la entidad certificadora. Si se prueba que el 
servidor es quien dice ser permite la conexión con la Web, en cualquier otro caso se lanza una advertencia al usuario 
indicando que el certificado es incorrecto.
	
En el archivo del certificado digital normalmente se incluyen los siguientes datos:

* **Número de serie:** utilizado para identificar el certificado.
* **Sujeto:** la entidad identificada (persona u organización)
* **Algoritmo de firma:** algoritmo utilizado para crear la firma.
* **Uso de la clave:** propósito de la clave pública (encriptación, firma, ambas).
* **Clave pública:** utilizada para encriptar un mensaje del sujeto o verificar su firma.
* **Emisor:** la entidad que expande el certificado.
* **Fecha de inicio de validez:** fecha a partir de la que el certificado es válido.
* **Fecha de fin de validez:** fecha en la que expira el certificado.
* **Algoritmo de huella digital:** el algoritmo utilizado para el hash del certificado.
* **Huella digital:** el hash del certificado utilizado para verificar que no se ha modificado el certificado.