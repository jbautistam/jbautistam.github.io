+++
title = "Distribuciones"
date = "2019-10-11"
description = "Preparación de scripts Spark SQL para ejecución sobre Databricks"
thumbnail = "/applications/baudbstudio/manual/025-distribuciones/025-distribuciones.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

**BauDbStudio** permite ejecutar scripts de Spark Sql contra un servidor ODBC en local o remoto pero
para trabajar con grandes cantidades de datos, podemos utilizar servicios como [Databricks](https://databricks.com/).
	
El problema es que **Databricks** no acepta los scripts SQL que tenemos hasta ahora, por tanto, debemos
transformarlo al formato de notebooks propio del servicio.
	
Así, por ejemplo, el contenido de un notebook para un script sencillo de Spark Sql sería así:

```SQL
-- Databricks notebook source

-- COMMAND ----------

-- MAGIC %md Crea la base de datos de cálculo

-- COMMAND ----------

CREATE DATABASE IF NOT EXISTS $dbcompute
	LOCATION '$mountpath/$dbcompute'
```

Vemos que los diferentes comandos (o celdas) del notebook deben separarse por sentencias **COMMAND** y que los comentarios
deben ir precedidos por **MAGIC \%md** para que se interpreten en formato Markdown.
	
Por supuesto, hacer estas modificaciones manualmente puede resultar muy trabajoso y llevarnos a errores, por eso en **BauDbStudio**
tenemos la posibilidad de crear distribuciones.
	
Para dar de alta una distribución, simplemente debemos elegir la opción de menú **Archivo | Nuevo | Distribución** o seleccionar
la opción **Nueva | Distribución** en el menú secundario del árbol de conexiones. Esto nos abrirá un formulario 
similar al que mostrábamos al principio de este artículo:

![Administración de distribuciones](/blog/applications/baudbstudio/manual/025-distribuciones/025-distribuciones.jpg "Formulario de administración de distribuciones")
		   
Aparte del nombre y la descripción, debemos seleccionar tanto el directorio en el que se encuentran nuestros scripts originales
escritos en Spark Sql y el directorio donde se dejarán los archivos traducidos al formato de notebooks de Databricks.
	
Además podemos controlar si en la traducción deseamos:

* Copiar los comentarios en los archivos de notebook o sólo los comandos sin comentarios.
* Pasar todos los nombres de archivos a minúsculas.
* Sustituir los argumentos utilizados en Spark Sql del tipo **$NombreArgumento** por la llamada a función **GetArgument("NombreArgumento")** que recomiendan en DataBricks.
		
Por último podemos seleccionar los valores que van a sustituir los parámetros fijos, es decir,
las constantes del tipo **{{Valor}}** que utilizamos al escribir nuestros [scripts de SQL](/blog/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql).

Una vez definida una distribución, podemos ejecutarla con la opción de menú **Distribuir** del menú secundario del árbol de conexiones:

![Distribuir scripts](/blog/applications/baudbstudio/manual/025-distribuciones/distribuir.jpg "Distribuir scripts")
		   
**Nota:** Aunque podemos ejecutar manualmente esta distribución y copiar los notebooks manualmente a un WorkSpace de Databricks, 
también existe una [consola](/blog/applications/baudbstudio/manual/130-consola-conversion-databricks/130-consola-conversion-databricks)
que puede integrarse en los pipelines de CI/CD para automatizar el proceso.