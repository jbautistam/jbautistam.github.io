+++
title = "DevConference para Android"
date = "2019-10-11"
description = "DevConference: visor para conferencias de desarrollo y programación"
thumbnail = "/applications/devconference-android/devconference-android.jpg"
tags = [ "Aplicaciones" ]
+++

Uno de mis proyectos personales del año pasado fue [DevConference](/blog/applications/devconference/devconference) con **UWP**:
una aplicación para recoger las conferencias y charlas sobre programación o desarrollo.
	
Este año he comenzado a estudiar **Xamarin Forms** y para afianzar los conocimientos una de mis tareas ha sido precisamente
convertir **DevConference** a una aplicación Android.
	
La forma de uso es prácticamente igual a la versión anterior, aunque alguna funcionalidad como la de abrir nuevas pistas
se ha perdido por el camino. El motivo es que simplemente quería ver cuánto me costaba hacer el cambio sabiendo que la arquitectura
ya estaba basada en MVVM.

Y finalmente ha sido más sencillo de lo que esperaba: simplemente crear las páginas utilizando Xamarin Forms, sustituir el manejo
de listas por el paquete de **FlowListView**, cambiar los repositorios internos, modificar
algunos métodos con await / async que han estado mal desde siempre y poco más.

Si queréis, podéis consultar la información sobre la aplicación en el [artículo anterior](/blog/applications/devconference/devconference) y
descargarla desde [Google Play](https://play.google.com/store/apps/details?id=com.bau.devconference).
Para aquellos que tengan tablet de Amazon, también está disponible en el **Amazon Store**.

La idea sigue siendo evolucionar la aplicación a algo un poco más avanzado y mantener una base de datos en la nube para compartir
conferencias pero tendrá que esperar hasta el año que viene.