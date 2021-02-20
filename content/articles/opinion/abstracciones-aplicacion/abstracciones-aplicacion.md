+++
title = "Abstracciones de aplicación"
date = "2020-11-01"
description = "Algunas ideas sobre las abstracciones en las aplicaciones"
thumbnail = "/articles/opinion/abstracciones-aplicacion/abstracciones.jpg"
tags = [ "Opinión" ]
+++

Cuando te pasas mucho tiempo en consultoría, tienes la oportunidad de acumular diferentes tipos de proyectos. No hay
nada mejor que llegar a un proyecto, comenzar a leer el código fuente y explicar a tu nuevo equipo porqué lo están haciendo mal.

**Nota:** supongo que el mismo proceso ocurriría cuando alguien me sustituía, maldiciones varias y el momento incómodo
de explicar porqué lo estaba haciendo mal.

El caso es que, entre todos los proyectos que desarrollé en los casi veinte años que me dediqué a la consultoría, hay
tres que recuerdo con una mezcla a partes iguales de miedo y orgullo: una aplicación de CRM para una empresa de telemárketing,
una aplicación de análisis médicos y otra de gestión de créditos hipotecarios.

A priori os preguntaréis que tienen en común todas ellas, en el momento que llegué: la falta de abstracción.

Todas ellas tienen un punto en común: cada proceso de aplicación es diferente en algún punto, no es lo mismo hacer un análisis de HCG que de EPO,
no es lo mismo un proceso de venta telefónica para un dispositivo móvil que para el alquiler de una casa y cada proceso hipotecario
de cualquier banco que elijáis es diferente en algún punto.

¿Y a qué nos lleva ésto cuando hablamos de programación? Nos lleva a hacer soluciones diferentes para cada caso.

En el caso de los análisis médicos, por ejemplo, había una aplicación que se encargaba de hacer hemogramas, otra para análisis de HCG,
otra para análisis de EPO,... ¿veis el patrón?

Al final, todas esas miniaplicaciones tenían mucho en común. De hecho, cada nuevo tipo de análisis que llegaba a desarrollo nos obligaba
a copiar la aplicación anterior y a modificarla en algún punto.

Esto nos lleva a muchos problemas: la aplicación C se ha copiado de la B que a su vez se copió de la A. 

¿Qué ocurre cuando nos encontaramos un error en la aplicación A? Que debemos corregir ese error en la aplicación B y C. 
Por supuesto, en mantenimiento, muchas veces alguien se olvidaba de cambiar la C (o la D o la E), muchas veces ni siquiera sabíamos que C existía.

Otro problema es añadir funcionalidades: si había algo que nos gustaba especialmente de la aplicación C lo podíamos añadir a la aplicación B y a
la A, o no. Dependía mucho del momento en el tiempo en que se hubiesen programado y del tiempo que tuviésemos para desarrollo. Por supuesto, 
esto también era un esfuerzo considerable y siempre alguna aplicación se quedaba desfasada en cuanto a funcionalidades y había que reprogramarla
(o copiar y pegar desde D y después modificarla para transformarla en A).

¿A dónde voy con todo ésto? A que el problema es de base, de no haber pensado en una abstracción que manejase todos los casos (o al menos el 90% de los
casos).

Y aquí vamos a lo que tienen estas aplicaciones en común: todas ellas se definen a partir de pasos básicos.

El caso de la gestión hipotecaria es quizá el más sencillo de entender: llega un cliente que quiere una hipoteca, deja una serie de documentación, se
gestionan los riesgos, se solicita documentación adicional a diferentes organismos (como las notas simples), se crea una cita con el notario, se firma,
se realizan los asientos en el registro, se pagan impuestos y se da por cerrada la hipoteca.

Por supuesto, cada banco lo hace de forma diferente, solicita diferente documentación y en diferente momento, envía correos electrónicos informativos o no, pasa la gestión
por diferentes departamentos, etc...

Pero al final, si los buscamos, tenemos pasos, por ejemplo:

1. Obtener documentación
2. Solicitar cita con el notario
3. Pagar impuestos...

Y un flujo de trabajo, dependiente del banco, que no es más que una composición de pasos y resultados.

Porque el flujo de trabajo depende de lo que pasa en cada momento. Cada paso puede tener diferentes respuestas: por ejemplo, si llamamos al notario
para pedir cita, tendremos dos respuestas diferentes, que nos dé una cita o que nos pida que llamemos dentro de una semana. Esto nos dirige el flujo a puntos
diferentes: establecer una cita en el calendario con el notario o crear un nuevo paso de solicitar cita para dentro de una semana.

¿Y en el caso de los análisis médicos? En realidad también son pasos: llega una muestra de sangre / orina, se apuntan los diferentes análisis que hay que hacer a la muestra,
si hay que hacer un HCG: se pasa por las diferentes máquinas, se apuntan los niveles de HCG (lo siento, el proceso de HCG era de los más sencillos pero lo suficientemente
aburrido como para no recordarlo), si hay que hacer un EPO: se hace la fase 1, si se encuentran niveles extraños se envía la muestra a otro laboratorio...

¿Observáis el patrón? pasos. Con diferentes resultados pero pasos.

Así que la solución para gestionar el caos de estos tres proyectos fue la misma: encontrar los pasos únicos y desarrollar un motor que los gestionase. Por supuesto
era algo más complicado, aunque como podréis suponer, la mayor parte del trabajo es en el interface de usuario y en la grabación de resultados, el motor no es más que una
máquina de estados simple.

¿Qué pasa ahora cuando queremos añadir un tipo de paso como por ejemplo enviar un correo electrónico? Definimos un tipo nuevo de paso, lo añadimos a nuestro flujo de trabajo y lo
asociamos a los diferentes clientes / procesos. En ocasiones tendremos que modificar el motor (en este caso para enviar correos, por ejemplo), la mayoría de las veces tendremos que 
modificar parte de los interfaces de usuario, pero no todas y cada una de nuestras aplicaciones.

¿Qué ocurre cuando hay un error en el motor? Básico, simplemente tenemos que corregir ese error en el motor, no en las aplicaciones (de hecho, sólo tenemos una aplicación).

Por supuesto, todo ésto lleva una implicación adicional: si queremos que esto funcione además de la aplicación que van a utilizar los usuarios finales tenemos que tener una
aplicación de administración de todos los parámetros y/o pasos del motor.

¿Por qué? Porque no queremos que sean nuestros desarrolladores quienes modifiquen el motor. Debemos intentar siempre que sean los usuarios finales o un departamento especializado
quien administre estos pasos sin necesidad de ir a la base de datos o a nuestros archivos de configuración a modificar manualmente estos datos.

Esto nos lleva a alguna complicación adicional, tenemos dos proyectos (o dos aplicaciones aunque no necesariamente) uno de administración y otro de gestión. Me he encontrado
en ocasiones que la aplicación de administración es más compleja que la propia aplicación de gestión, con pasos muy complicados de modelar.

Tenemos en esos casos la tentación de no administrar, de dejar esos pasos para que se modelen manualmente: no lo hagas. Nunca. Punto.

Todo el tiempo que inviertas en la administración y el modelado de tus procesos será tiempo que tu equipo de desarrollo no invertirá en la propia administración del proyecto
y que podrás delegar en usuarios no técnicos. Todo ese tiempo invertido en crear una abstracción en tus modelos será tiempo que ahorrarás en el futuro. Evita los
procesos manuales y copiar y pegar código, créeme, tus usuarios te lo agradecerán a la larga y tú podrás dedicarte a otras cosas. 

Recuerda siempre que el trabajo fundamental de desarrollo no es escribir código, es pensar en cómo tiene que ser el proceso, es descubrir los patrones que te permitirán
desarrollar menos.

En todos estos casos, el proceso era común y aunque a primera vista parecieran desarrollos diferentes, demostramos siempre que podíamos abstraer una base común. 

Cuando adoptamos un sistema de pasos (que por cierto está inventado desde hace muchos años, se llama 
[BPM](https://es.wikipedia.org/wiki/Gesti%C3%B3n_de_procesos_de_negocio)), nuestro desarrollo
cambió por completo, pasamos de corregir errores en diferentes aplicaciones a una única aplicación y en lugar de copiar / pegar aplicaciones / clases pasamos a un proceso de
gestionar pasos y ejecutar esos pasos. Nuestra vida cambió.

¿Crees que no se aplica a tu aplicación? ¿Que no hay una abstracción común? Mira a tu alrededor, siempre la hay, debes buscarla con atención.

Si estás copiando y pegando clases para importar diferentes CSVs, por ejemplo, te vas a encontrar un problema cuando te cambien el tipo de archivo de entrada a Parquet porque el
departamento de negocio se ha dado cuenta que este tipo de archivos es más pequeño y por tanto disminuye los costes.

En ese caso, vas a tener que cambiar todas esas clases que has copiado y pegado.

¿Por qué no te decides por un sistema más sencillo? Algo que defina tus archivos, por ejemplo, como campos y tipos y como tabla de base de datos final. Guarda estas 
estructuras en archivos XML por ejemplo o en un YAML o en un JSON o como registros de tu base de datos, como prefieras y crea una aplicación que lea esta configuración
y las grabe en tablas de datos. No necesitas clases para todo.

¿Estás haciendo un montón de informes diferentes cada uno de ellos con sus propias características y clases? Crea una estructura que identifique las columnas de salida,
los posibles filtros y la SQL y un motor de gestión de informes que lea y presente estas estructuras. Al fin y al cabo es lo que hacen las aplicaciones de BI tipo
Tableau desde hace años.

No digo que sea sencillo, nunca lo es, pero recuerda: el tiempo de abstracción, de sentarte a pensar, de modelar en papel, es tiempo bien gastado. No obligues a tu
equipo a copiar y pegar, obliga a tu equipo a pensar, es de las pocas cosas en esta vida que no provoca ninguna enfermedad.