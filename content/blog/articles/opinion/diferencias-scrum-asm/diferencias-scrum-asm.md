+++
title = "Diferencias entre Scrum y ASM"
date = "2019-10-11"
description = "Las diferencias existentes entre las metodologías Scrum y ASM"
thumbnail = "/articles/opinion/diferencias-scrum-asm/diferencias-scrum-asm.jpg"
tags = [ "Opinión" ]
+++

Al fin ha ocurrido. Tenía que pasar antes o después y ha sido hoy. Por fin el cliente para el que trabajo en este proyecto
ha decidido dejar la tecnología de gestión de ASM y se pasa a la metodología ágil, concretamente a Scrum.

Debería estar contento, siempre he defendido este tipo de técnicas, sin embargo, lejos de alegrarme,
me plantea ciertas dudas.

La tecnología que utilizábamos hasta ahora, ASM, es bastante sencilla. Para quien no la conozca, ASM significa 'A Su Manera'. 
También se la conoce coloquialmente como ASB o 'A Su Bola'. Consiste en que cada programador recibe un documento de requisitos bastante 
esotéricos si tiene suerte o un resumen de reunión del tipo "lo que tienes que hacer es..." si no tiene 
tanta suerte. Una vez definido el requisito, se busca la vida para desarrollarlo.

La técnica como podemos imaginar, ha dado resultados dispares, desde grandes éxitos como el formularlo de entrada 
al sitio hasta grandes fracasos como el sistema de facturación. Opinamos que tanto unos como otros dependen más de 
la experiencia del programador que de la tecnología en sí misma así que hemos decidido cambiarla.

Mi problema no viene tanto por Scrum como por las razones esgrimidas para venderlo:

* Evitaremos la gestión de requisitos.
* El tiempo de desarrollo será menor.
* Tendremos el proceso más controlado.
	
Y aquí empiezan mis problemas. ¿Cómo explicar dentro de unos meses que esas afirmaciones son incorrectas por no decir mentira?

* Ni Scrum ni ninguna metodología ágil elimina la gestión de requisitos. No puedo desarrollar si no sé qué 
tengo que desarrollar: es así de simple. Es cierto que las métodologías ágiles al tener iteraciones de un par 
de semanas facilitan la gestión simplemente porque nos da tiempo a hacer menos cosas. Por tanto nos centraremos en 
veinte requisitos y no en dos mil, pero sigue siendo gestión de requisitos.

* El tiempo de desarrollo no es menor. El tiempo será el mismo. Se elimina, eso sí, el tiempo de análisis de todo el 
proyecto y se divide en microanálisis pero el tiempo es el mismo. Admito que la sensación de desarrollo rápido para el 
usuario es mayor dado que cada dos semanas obtiene código nuevo y probado pero el tiempo no cambia.

* El proceso es tan controlado o descontrolado como ASM/ASB. El control no depende de la técnica si no de las personas 
y aunque Scrum ofrezca métodos para observar el proceso, si nadie lo vigila, el control o descontrol será el mismo.

Así que, como se puede ver mi gran duda se deriva de la similitud existente para un "no técnico" entre Scrum y ASM y los 
vicios adquiridos hasta ahora que pueden dar al traste con nuestros esfuerzos y transformar Scrum en algo que no es realmente
ni ágil ni Scrum.
	
Porque además me temo que hay cosas que nadie se ha atrevido a explicarle al cliente:

* Los requisitos de un sprint no se pueden modificar: ni se eliminan ni se añaden tareas una vez arrancado. Es decir, 
se acabó el "esto es urgente", "me puedes hacer el favor de..." o similares. Es lo que hay. 

* Vamos a hacer lo que hay en el backlog del sprint. Lo vas a tener en quince días tal y como lo pediste. Esto implica que más vale 
que te pienses bien lo que pides porque no hay vuelta atrás.

* Los programadores tienen poder de decisión. Si dicen que un desarrollo son tres días: es inapelable. Tienen sus razones. 
Si dicen que un desarrollo no cabe en un sprint, es por algo. No les discutas y piensa en cómo dividir la tarea o asigna más personas
a la iteración (que aunque parezca mentira puede implicar más tiempo, lo del mes-hombre sigue siendo una falacia por mucho que
cambie la tecnología).

* El equipo de desarrollo antes o después va a querer solucionar algo que denominamos **deuda técnica** inherente a la programación 
con iteraciones. Esta deuda técnica se comerá parte de tu backlog cada cierto tiempo. Lo hacemos por el bien de la aplicación 
y tendrás que vivir con ello.

* Hay ciertos procesos que necesitamos y que nunca tienes en cuenta: hablamos de infraestructura, de log, de patrones de 
repositorio, de refactorización, de pruebas unitarias y un montón de cosas que como cliente no te has parado a pensar. Sabemos 
lo que hacemos, hemos estudiado mucho y el desarrollo es nuestro trabajo así que nos vamos a reservar tiempo en el backlog 
tanto para estas cosas como para resolver incidencias y hacer cambios estructurales. No nos discutas, recuerda que miramos por tu negocio.

* Necesitamos requisitos, pero no sólo eso, necesitamos respuestas y no pueden ser dentro de dos días, tienen que ser ya, en el 
momento en que nos surja una duda. Eso quiere decir que necesitamos alguien a quien preguntar con poder de decisión casi absoluto. 
Nos da igual quien. Somos gente afable... tú decides.

* Planifícate porque el equipo va a tener reuniones a diario. Son cortas (excepto las de planificación o retrospectiva) pero necesarias 
y no se posponen. Tú traes el café o el té, nosotros llevamos los bollos.

Y sin esas premisas y considerando la experiencia acumulada en estos años, miedo me da. Os mantendré informados. Los próximos meses prometen 
diversión para todos.