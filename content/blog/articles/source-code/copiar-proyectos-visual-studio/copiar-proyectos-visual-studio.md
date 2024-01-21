+++
title = "Aplicación para copiar proyectos de Visual Studio"
date = "2019-10-11"
description = "Código fuente de una aplicación para copiar proyectos de Visual Studio"
thumbnail = "/articles/source-code/copiar-proyectos-visual-studio/copiar-proyectos-visual-studio.jpg"
tags = [ "Programación" ]
+++

Dado que siempre tengo una nueva aplicación entre manos, siempre acabo con soluciones de
**Visual Studio** compuestas por montones de proyectos compartidos entre sí.
De hecho, mi récord está en los 77 proyectos que actualmente componen 
[BauPlugStudio](http://bauplugstudio.webs-interesantes.es/).
	
Eso no me causa ningún problema hasta el momento en que quiero compartirlos o subirlos como una aplicación
a **GitHub**. En esos casos tengo que ir copiando todos los proyectos en una carpeta, asociarlos
a la misma solución y cambiar las referencias.
	
Como soy un fiel seguidor del credo político de Anguita y su 'Programa, programa, programa' me he creado
una aplicación que a partir de un archivo de solución copia todos los proyectos en un directorio y 
les cambia las referencias.
	
Lo podéis encontrar en **GitHub** en [CopySolutionsVisualStudio](https://github.com/jbautistam/CopySolutionsVisualStudio).