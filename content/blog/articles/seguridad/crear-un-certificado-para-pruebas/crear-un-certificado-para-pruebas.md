+++
title = "Crear un certificado para pruebas con makeCert"
date = "2019-10-11"
description = "Utilización de makeCert para crear un certificado para pruebas"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación", "Seguridad" ]
+++

Los [certificados digitales](/blog/articles/seguridad/certificados-digitales/certificados-digitales) nos permiten realizar
comunicaciones seguras utilizando [criptografía asimétrica](/blog/articles/seguridad/clave-publica/clave-publica).
	
Los **certificados digitales** los emite una **entidad certificadora** que recoge nuestros datos y previo pago
nos devuelve un archivo que podemos utilizar en nuestro trabajo.
	
Cuando estamos desarrollando muchas veces no deseamos asumir ese coste para las máquinas de desarrollo o preproducción
que vamos a utilizar en las pruebas, en estos casos podemos utilizar la herramienta **makeCert** para la creación de 
un **certificado** de prueba.
	
Para crear el **certificado** debemos seguir los siguientes pasos:

1. Abrir una ventana de comandos de **Visual Studio** (normalmente está en el menú de inicio de Windows en 
*Visual Studio Tools | Visual Studio 8 Command Prompt*).
		
2. Desde la ventana de comandos ejecute la siguiente instrucción:

```
makecert -n "CN=NombreTest" -r -sv TestCA.pvk TestCA.cer
```

Esta instrucción crea el archivo que podemos utilizar como **certificado**. En esta herramienta
utilizamos los parámetros:

* `-n` nombre del titular del CA raíz.
* `-r` indica que es un certificado autofirmado. Normalmente el nombre se antecede del prefijo *CN #.
* `-sv` indica que el certificado contiene la clave privada del **certificado**.
* `TestCA.cer` es el nombre del archivo que contiene la clave pública del certificado.
* `TestCA.pvk` es es el archivo donde se encuentra la clave privada.
			
Cuando aparezca el cuadro de diálogo *Crear contraseña de clave privada* introduzca la contraseña y su
confirmación y pulse el botón *Aceptar*. Se puede pulsar el botón *Ninguna* para no asignar 
contraseña aunque por supuesto no se recomienda.

![Cuadro de diálogo de creación de contraseñas de clave privada](/blog/articles/seguridad/crear-un-certificado-para-pruebas/crear-contrasena-de-clave-privada.jpg "Cuadro de diálogo de creación de contraseñas de clave privada")
				 
Una vez introducida la clave privada, aparece el cuadro de diálogo *Escriba la contraseña de la clave
privada* donde debemos volver a teclear la clave privada y pulsar el botón *Aceptar*.

![Cuadro de diálogo Escribir contraseña de clave privada](/blog/articles/seguridad/crear-un-certificado-para-pruebas/escribir-contrasena-de-clave-privada.jpg "Cuadro de diálogo Escribir contraseña de clave privada")

Así tendremos por fin nuestro **certificado de prueba**.