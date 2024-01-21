+++
title = "Scripts de ETL"
date = "2019-10-11"
description = "Manual de creación de scripts de ETL (Extract Transform Load) con BauDbStudio"
thumbnail = "120-scripts-etl.jpg"
tags = [ "Aplicaciones", "BauDbStudio" ]
series = ["BauDbStudio"]
+++

Al trabajar con datos, normalmente nos encontramos con la necesidad de ejecutar procesos repetitivos como copiar
archivos, realizar consultas, copiar datos resumidos entre base de datos, etc...
	
Este tipo de trabajos se conocen como procesos ETL (Extract, Transform, Load). **BauDbStudio** permite la definición
y ejecución de este tipo de procesos.

## Tipos de procesos ETL

Para trabajar con procesos ETL, **BauDbStudio** define diferentes procesadores, cada uno de estos procesadores se encarga
de un tipo de trabajo. 
	
Actualmente podemos definir procesos de:


* **[Tratamiento de archivos](/blog/applications/baudbstudio/manual/120-scripts-etl/005-etl-archivos/005-etl-archivos)**:
copia, borrado de archivos, ejecución de comandos del sistema operativo, transformación de archivos (CSV, Excel, Parquet).
* **[Tratamiento de archivos en la nube](/blog/applications/baudbstudio/manual/120-scripts-etl/010-etl-cloud-storage/010-etl-cloud-storage)**:
tratamiento de archivos en sistemas de almacenamiento en la nube (actualmente Azure Storage Blob),
sobre todo para descarga y subida de archivos.
* **[Procesos REST](/blog/applications/baudbstudio/manual/120-scripts-etl/020-etl-rest/020-etl-rest)**:
llamadas a procesos REST (actualmente en desarrollo).
* **[Procesos de base datos](/blog/applications/baudbstudio/manual/120-scripts-etl/025-etl-base-datos/025-etl-base-datos)**:
procesos de consulta, copia y conversión de datos entre bases de datos relacionales.

## Definición de procesos

**BauDbStudio** define los procesos de ETL utilizando diferentes archivos XML para cada tipo de proceso.

Para dirigir el proceso, se utilizan dos archivos especiales: 

* **Archivo de proyecto:** que indica el orden de los pasos a ejecutar.
* **Archivo de contexto:** que mantiene los parámetros como por ejemplo las cadenas de conexión o
nombres de directorio. Este tipo de archivos nos permite ejecutar el mismo proceso pero con datos 
diferentes 

Cada tipo de proceso tiene sentencias y esquemas diferentes que se explica en su propia página.

* Información del [archivo de proyecto](/blog/applications/baudbstudio/manual/120-scripts-etl/000-etl-proyecto/000-etl-proyecto)
* Información del [archivo de contexto](/blog/applications/baudbstudio/manual/120-scripts-etl/002-etl-contexto/002-etl-contexto)
* Información de procesos
	* [Procesos de archivos](/blog/applications/baudbstudio/manual/120-scripts-etl/005-etl-archivos/005-etl-archivos)
	* [Procesos sobre archivos en la nube](/blog/applications/baudbstudio/manual/120-scripts-etl/010-etl-cloud-storage/010-etl-cloud-storage)
	* [Procesos REST](/blog/applications/baudbstudio/manual/120-scripts-etl/020-etl-rest/020-etl-rest)
	* [Procesos de base datos](/blog/applications/baudbstudio/manual/120-scripts-etl/025-etl-base-datos/025-etl-base-datos)

## Ejecución de procesos de ETL

Para ejecutar los procesos ETL, si nos colocamos sobre un [archivo de proyecto](/blog/applications/baudbstudio/manual/120-scripts-etl/000-etl-proyecto/000-etl-proyecto)
en el árbol de archivos y pulsamos la opción de menú **Ejecutar**:
	
![Menú de ejecución de procesos ETL](/blog/applications/baudbstudio/manual/120-scripts-etl/menuejecucionetl.jpg "Menú de ejecución")
			
Se nos abrirá la ventana de ejecución de **procesos ETL**:

![Ejecucion de procesos Etl](/blog/applications/baudbstudio/manual/120-scripts-etl/ejecucionetl.jpg "Ejecucion de procesos Etl")
			
En la sección izquierda vemos el contenido del [archivo de proyecto](/blog/applications/baudbstudio/manual/120-scripts-etl/000-etl-proyecto/000-etl-proyecto).

En la parte derecha debemos seleccionar el [archivo de contexto](/blog/applications/baudbstudio/manual/120-scripts-etl/002-etl-contexto/002-etl-contexto)
con el que deseamos ejecutar el proceso:
	
![Ejecución de procesos Etl sobre un contexto](/blog/applications/baudbstudio/manual/120-scripts-etl/ejecucionetlcontext.jpg "Ejecución de procesos Etl sobre un contexto")
			
Una vez seleccionado un contexto correcto, podemos pulsar sobre el botón **Ejecutar** para procesar los diferentes pasos del script.

Existe una [consola](/blog/applications/baudbstudio/manual/140-consola-ejecucion-etl/140-consola-ejecucion-etl)
asociada a **BauDbStudio** que nos permite ejecutar los proyectos por separado.