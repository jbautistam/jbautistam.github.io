+++
title = "Consideraciones sobre DateTime en SQLServer con respecto a .Net"
date = "2019-10-11"
description = "Algunas consideraciones que debemos tener en cuenta cuando comparamos los datos de un dateTime de .Net con un DateTime de SQL Server"
thumbnail = "/articles/development/datetime-sqlserver/datetime-sqlserver.jpg"
tags = [ "Programación" ]
+++

Tiendo a pensar que cuando dos elementos tienen el mismo nombre, significan lo mismo, pero en desarrollo
esto no tiene porqué ser así.

Tenemos una de esas diferencias entre la estructura `DateTime` de.Net y el tipo de datos
`DateTime` de SQL Server.

Si por ejemplo grabamos en un registro de base de datos un valor `DateTime` de.Net en un campo `DateTime`
de SQL Server, lo cargamos de nuevo y los comparamos nos encontraremos que en muchas ocasiones no son iguales.

El problema radica en que la precisión de ambos tipos de datos es distinta. Así la definición de `DateTime`
en .Net es la siguiente:

> Time values are measured in 100-nanosecond units called ticks, and a particular date is the number of ticks 
> since 12:00 midnight, January 1, 0001 A.D. (C.E.) in the GregorianCalendar calendar.

Mientras que el tipo de datos `DateTime` de SQL Server es:

> Accuracy – Rounded to increments of.000,.003, or.007 seconds

Eso quiere decir, que no podemos hacer una comparación ni siquiera a nivel de milisegundos dado que la grabación
en la base de datos de.008 milisegundos se redondea al valor superior. Si no me equivoco con las matemáticas, 
esto quiere decir que sólo podemos hacer comparaciones entre ellos a nivel de décimas de segundo que para algunas
aplicaciones puede no ser suficiente.

La otra diferencia fundamental es el valor mínimo o máximo de las fechas. En .Net el valor mínimo es el día 1-1-1 00:00:00
mientras que en SQL Server, el valor mínimo es del 1-1-1753 00:00:00.

Si por tanto, intentamos grabar `DateTime.MinValue` sobre un registro de base de datos, obtendremos una excepción
de tipo "out of range" bastante desagradable.

De hecho, SQL Server tiene varios tipos de datos para las fechas, cada uno con un tamaño y una precisión diferente:

![Tipos de fechas en SQLServer](/blog/articles/development/datetime-sqlserver/tipos-date-sqlserver.jpg "Tipos de fechas en SQLServer")
		
La recomendación es utilizar `DateTime2` si queremos mayor precisión pero debemos tener en cuenta que ciertas
funciones como `GetDate`, según la versión, puede que sigan utilizando tipos de datos `DateTime` aunque
los grabemos en un campo `DateTime2`.
