+++
title = "La inseguridad de los certificados digitales"
date = "2019-10-11"
description = "Consejos sobre seguridad y certificados digitales para directivos"
thumbnail = "/articles/opinion/inseguridad-certificados-digitales/inseguridad-certificados-digitales.jpg"
tags = [ "Opinión", "Seguridad" ]
+++

![](/blog/articles/opinion/inseguridad-certificados-digitales/inseguridad-certificados-digitales.jpg)

Vamos a comenzar un proyecto nuevo con una empresa externa. Es sencillo: un servicio REST que envía
y recibe documentos firmados y encriptados digitalmente con un 
[certificado digital](/blog/articles/seguridad/certificados-digitales/certificados-digitales).
	
En la última reunión, la susodicha empresa externa le pasó a mi jefe una copia en un pendrive del certificado que vamos
a utilizar. Mi jefe, que nunca deja de sorprenderme, me lo envió adjunto en un correo electrónico y puso en
copia a dos personas de negocio y a su secretaria, supongo que para que tuvieran constancia de que el proyecto
estaba a punto de empezar.
	
Debió pensar que era una buena idea pero la realidad es que ha puesto en riesgo todo el proyecto. A
partir de este momento podemos considerar que el certificado digital ya no tiene nada de privado.
	
Y no es porque mis compañeros de negocio formen parte de una red de cibercriminales o pretenda extorsionar
a la empresa ni porque el departamento de sistemas no cumpla con todos los protocolos de seguridad y tenga 
*correctamente* asegurado el correo electrónico. Es porque nuestra superficie de ataque hace tiempo que 
ha salido del ámbito de la propia empresa.
	
Me explico: ahora mismo mis compañeros pueden acceder al correo electrónico desde el ordenador de su casa, el mismo
ordenador que utilizan de vez en cuando para hacer sus presentaciones y donde conectan pendrives de procedencia
dudosa. Además, todos ellos tienen acceso al correo electrónico desde sus móviles y tablet. 
	
El problema es que ninguno de estos dispositivos está dentro del paraguas de seguridad de la empresa. Alguno de
ellos cree que el antivirus no sirve para nada excepto para consumir tiempo de proceso del ordenador y por tanto
no lo han instalado. Aún peor, quien lo tiene en su móvil ha decidido instalar uno de esos 
*antivirus gratuitos* que nadie sabe qué hace realmente.
	
Con estos mimbres, es bastante posible que alguno de ellos haya caído en manos de un keylogger o reutilice la contraseña
de Facebook / Twitter para el trabajo y entonces todo está perdido. Si yo fuese uno de los *chicos malos*, seguramente me
interesase por esos archivos con extensión `cer`, `der` o similar.
	
Y aquí vienen mis consejos. De verdad son fáciles:

* No es necesario que te preocupes por tu seguridad: ya habrá alguien que lo hará. Posiblemente sean 
los *chicos malos# pero menos es nada.

* Tus datos sólo son privados ahora. En cuanto dejen de ser privados, olvídate de ellos, pasan a ser públicos para
siempre.
		
Y ahora en serio:

* No compartas información de seguridad por medios inseguros. Y considera inseguro el correo electrónico y los programas
de mensajería. De hecho, considera inseguro tu ordenador.

* Tu información de seguridad sólo le interesa a los responsables de seguridad o de sistemas de tu empresa. 
Absolutamente a nadie más, ni siquiera a tus desarrolladores. A tus desarrolladores les interesan los certificados
para desarrollo y estos deben ser diferentes a los de producción.
		
Es sencillo ¿verdad? Tomemos nota.
	
**Nota:** habrá quien se pregunte porqué el artículo tiene como cabecera una imagen de **Ashley Madison**:
fijaos en la parte inferior de la imagen. Promete todas las medidas de seguridad y fiabilidad, de hecho, 
tiene incluso el icono de SSL bien visible. Todos sabemos de qué sirvieron esas medidas. 

Con tres palabras **de absolutamente nada**.