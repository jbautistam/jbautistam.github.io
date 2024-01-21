+++
title = "Almacenamiento de contraseñas de usuario"
date = "2019-10-11"
description = "¿Cómo debemos almacenar las contraseñas de nuestros usuarios en archivos y bases de datos?"
thumbnail = "/articles/seguridad/almacenamiento-contrasenas/almacenamiento-contrasenas.jpg"
tags = [ "Programación", "Seguridad" ]
+++

Una de las necesidades básicas de los sistemas de seguridad es el almacenamiento de contraseñas
de usuario en bases de datos o sistemas de archivos.
	
La principal dificultad en este caso, es impedir que un atacante que obtenga los datos de una tabla de usuarios
y contraseñas pueda descifrar la clave de un usuario y suplantar así su identidad.
	
Para almacenar esas tuplas de usuario y contraseña existen varias técnicas, desde guardarlas como 
texto plano a almacenarlas utilizando un sistema de cifrado o hash.
	
Por supuesto, ningún programador que se precie debería almacenar las contraseñas en texto plano:
en ese caso cualquier robo de información pondría en riesgo la identidad del usuario y además
es posible que incumplamos ciertas leyes como la LORTAD.
	
La segunda opción es encriptar la clave almacenada utilizando sistemas como AES o DES pero
nos accarrea ciertos problemas. El primero de ellos es que debemos almacenar la clave de cifrado,
algo que también puede ser susceptible de ataque.
	
El segundo problema es que una vez robados los datos, el atacante puede utilizar algoritmos de
fuerza bruta para inferir la clave de cifrado.
	
Pero quizá lo más importante en este tipo de casos es cómo transmite un usuario legítimo su contraseña
al sistema: si lo transmite como texto plano nos exponemos a ataques de Man in the middle, si lo transmite
cifrado debe conocer la clave que tenemos en nuestro sistema lo que nos expone a otras grietas en
nuestra seguridad. Por supuesto existen sistemas de clave pública / privada que atenuan este problema
pero añaden una capa de complejidad al proceso.
	
Por eso, lo más común es almacenar las contraseñas utilizando sistemas de hashing como MD5, SHA1, SHA256, SHA384,
	o RIPEMD-160.
	
**Nota:** ya hemos hablado en este sitio de cómo obtener los valores
[MD5](/blog/articles/seguridad/obtener-el-valor-md5-de-una-cadena/obtener-el-valor-md5-de-una-cadena)
y el [SHA1](/blog/articles/seguridad/obtener-valor-sha1/obtener-valor-sha1) de una cadena. El resto de sistemas
funcionan de una forma similar, por eso no se repite aquí ese código.
	 
En estos casos, al grabar un usuario / contraseña en la base de datos no grabamos realmente la contraseña del usuario
si no el resultado de la función Hash sobre la contraseña:

![Tabla de usuario y contraseña con MD5](/blog/articles/seguridad/almacenamiento-contrasenas/tabla-usuario-contrasena-md5.jpg "Tabla de usuario y contraseña con MD5")
			
Si ya tenemos el usuario y contraseña codificada en nuestra base de datos, para que un usuario pueda entrar al
sistema ya no debe enviarla en texto plano: debe enviar el resultado de la función hash de la contraseña introducida
en el sistema de login. Es decir, la contraseña nunca viaja en plano del cliente al servidor. 
	
![Envío del resultado de la función hash sobre la contraseña](/blog/articles/seguridad/almacenamiento-contrasenas/envio-hash.jpg "Envío del resultado de la función hash sobre la contraseña")

Tampoco el servidor conoce la contraseña del usuario dado que una función hash no se puede deshacer,
lo que se hace es comparar la cadena recibida con la cadena almacenada en el servidor cuando se dio de alta
el usuario.

Antes de empezar, deberíamos considerar cuál es la función hash más adecuada 
para nuestro caso particular. Tengamos en cuenta, eso sí, que MD5 se considera un sistema 
obsoleto en el que es sencillo obtener colisiones y deberíamos descartarlo. Además para SHA1 han comenzado a 
aparecer las [primeras colisiones](https://security.googleblog.com/2017/02/announcing-first-sha1-collision.html) 
y por lo tanto tampoco parece un sistema adecuado estos días:
	
![Colisiones con sistemas SHA1](/blog/articles/seguridad/almacenamiento-contrasenas/collision-sha1.jpg "Colisiones con sistemas SHA1")
		 		
Mi recomendación sería utilizar sistemas de hash más potentes como SHA128 y SHA256 aunque este
tipo de sistemas posiblemente sean más lentos a la hora de generar los resultados.

**Nota:** Aunque sabemos que MD5 es un sistema inseguro para el almacenamiento de datos sigue teniendo otros usos en la comparación de archivos por su velocidad. 
En sistemas donde la fiabilidad no es un requisito y no nos preocupen demasiado las colisiones, como en la comparación de imágenes, se sigue utilizando.

Uno de los problemas de almacenar las contraseñas de esta forma, es que cualquier atacante puede utilizar el mismo 
sistema para descifrarlas usando tablas rainbow. Una tabla rainbow es una tabla creada a partir de
una serie de contraseñas habituales o una lista de contraseñas asociadas a su cadena hash:

![Ejemplo de tabla rainbow](/blog/articles/seguridad/almacenamiento-contrasenas/tabla-rainbow.jpg "Ejemplo de tabla rainbow")
			
Con estas tablas, el atacante puede buscar la cadena hash almacenada en nuestra tabla y obtener la contraseña
inicial para suplantar al usuario.
	
Por eso, nunca deberíamos almacenar las contraseñas utilizando una simple transformación, deberíamos añadir
a la contraseña una cadena adicional (conocida como Salt o Inicialization Vector - IV). Esta cadena puede ser
cualquier cadena aleatoria independiente para cada usuario. Una vez obtenido el IV debemos almacenarlo en nuestra 
base de datos:

![Tabla de usuario / hash con IV](/blog/articles/seguridad/almacenamiento-contrasenas/tabla-iv.jpg "Tabla de usuario / hash con IV")
	
Al contrario que la contraseña, la cadena IV se puede almacenar en plano sin ningún problema puesto que su único
propósito es añadir entropía al resultado.
	
En este caso, el proceso es ligeramente diferente, el usuario nos envía el resultado de la función hashing de su contraseña
y nosotros almacenamos y comparamos con el resultado de calcular una función hash obtenida de concatenar la cadena IV 
con la cadena enviada por el usuario:
	
![Envío de hash utilizando IV](/blog/articles/seguridad/almacenamiento-contrasenas/envio-iv-hash.jpg "Envío de hash utilizando IV")

Es un poco más farragoso pero así evitamos los ataques de tablas rainbow y tenemos una aplicación mucho más segura.

**Nota:** En algunos casos he visto utilizar el usuario como cadena IV. Por supuesto, los 
ciberdelincuentes también conocen esta forma de trabajar y utilizan sistemas similares a los de las tablas rainbow 
para atacar el cifrado.
	
En este artículo sólo hablamos de la forma de almacenar la contraseña en nuestro sistema. La comunicación
de la contraseña con el usuario utilizando funciones hashing sigue siendo insegura ante ataques Man in the middle.
	
Cualquier atacante que obtenga la cadena hash que ha enviado el usuario al sistema, puede suplantar la identidad de
éste dado que sustituye a todos los efectos a la contraseña inicial. Existen técnicas para evitar este tipo de ataques
que dan para otro artículo.
	
Hasta entonces, espero que este artículo sirva para añadir un poco de luz sobre este tema y que pase a nuestro	
arsenal de técnicas de seguridad básica.
	
**Nota:** La imagen de cabecera de este artículo es una captura de la web
[Have I been pwned?](https://haveibeenpwned.com/) que mantiene
una lista de servicios que han tenido fallos de seguridad en los últimos años y donde podemos
consultar si nuestra dirección de correo electrónico está en riesgo.