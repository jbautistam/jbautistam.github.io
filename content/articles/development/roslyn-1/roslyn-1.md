+++
title = "Fundamentos de Roslyn"
date = "2019-10-11"
description = "Fundamentos de Roslyn, el servicio de compilación de .NET de Microsoft"
thumbnail = "/articles/development/roslyn-1/roslyn-1.jpg"
tags = [ "Programación" ]
+++

De todas las novedades que.NET ha incorporado a lo largo de los años una de las que más me ha llamado la
atención ha sido [Roslyn](https://github.com/dotnet/roslyn) por la forma 
en que rediseña el modo de comunicarnos con los compiladores y los nuevos servicios que nos ofrece.

**Roslyn** ya lleva unos años con nosotros, de hecho, su primera versión se podía descargar como extensión de
Visual Studio 2010 SP 1 a partir de Octubre del 2011 aunque no fue hasta el 2012 cuando se consideró la API completa
para el desarrollo de compiladores. Por último, en el Build 2014 Microsoft liberó el proyecto **Roslyn**
como open source y lanzó la integración con Visual Studio 2013. A partir de ese momento otras compañías como
Xamarin comenzaron a adoptar esta tecnología para sus propias herramientas.

El propio **Anders Hejlsberg** en el [Build 2011](https://channel9.msdn.com/Events/BUILD/BUILD2011/TOOL-816T)
introducía **Roslyn** como un proyecto a largo plazo y definía un compilador clásico como una caja negra en la que se introducía
código fuente y se obtenía código objeto. **Roslyn** sin embargo se define como un 'servicio de compilación'
aunque debemos entender aquí 'servicio' como API y no como una especie de 'compilación en la nube' o SaaS. Es decir,
**Roslyn** nos ofrece una serie de métodos para acceder al proceso de compilación y al código fuente.

Pero ¿para qué nos interesan estos servicios de compilación? ¿para qué romper la estructura de caja negra de un compilador
y ofrecérsela a los desarrolladores? Es simple, para facilitar su trabajo.

Quizá no nos demos cuenta pero muchas de las herramientas que empleamos diariamente en nuestros editores están íntimamente
relacionadas con los procesos de compilación. Sin ir más lejos, el propio **Intellisense** accede al código fuente para 
mostrarnos las variables o las propiedades y métodos de un objeto (de ahí el molesto 'Se está construyendo aún la información
de la librería' que aparece de vez en cuando) o para subrayarnos en rojo los errores sintácticos. 

Pero tenemos más herramientas que acceden al código fuente y que interpretan este código: los analizadores estáticos, las herramientas
de análisis de métricas de código, las ventanas de explorador de clases, la búsqueda de referencias, CodeLens e incluso herramientas externas como 
[ReSharper](https://www.jetbrains.com/resharper/) o 
[NDepend](www.ndepend.com/) leen el código fuente que estamos escribiendo para ofrecernos su
ayuda.

Hasta la aparición de **Roslyn** si queríamos hacer alguna extensión como por ejemplo un documentador, debíamos escribir nuestro
propio intérprete o parser de código fuente. Con **Roslyn** podemos simplemente acceder a sus servicios, de hecho, 
escribir un analizador de código simple con **Roslyn** no nos llevará más de unos cientos de líneas
(pueden leer un ejemplo en [Use Roslyn to Write a Live Code Analyzer for Your API](https://msdn.microsoft.com/en-us/magazine/dn879356).

Pero **Roslyn** nos ofrece otras funcionalidades como generar el código de una aplicación y compilarlo en ejecución sin tener
que recurrir **Emit**, para ejecutar C# como script del mismo modo que la nueva consola de REPL (Read-Eval-Print-Loop) de Visual Studio 2015 
Update 1 transformandoC# en un lenguaje dinámico como Python o traducir código de un lenguaje a otro como C# a Visual Basic o viceversa. 
Además, al separar el compilador en servicios nos abre posibilidades como generación de nuevos lenguajes para .NET o incluso la
tan comentada compilación a código nativo de estos últimos meses. 

En próximos artículos veremos cómo sacar partido de **Roslyn** y algunas de las posibilidades que nos ofrece.