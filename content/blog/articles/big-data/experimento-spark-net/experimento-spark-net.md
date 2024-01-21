+++
author = "Jose Antonio Bautista"
title = "Experimentando con Spark.Net"
date = "2020-10-10"
description = "Mi primer experimento utilizando Spark.Net en local"
thumbnail = "articles/big-data/experimento-spark-net/images/Experimento-Spark-Net.jpg"
tags = [ "spark", ]
categories = [ "spark" ]
+++

Sé que últimamente lo digo mucho, pero estoy aprendiendo a programar con Spark utilizando sobre todo
Spark Sql y algo de PySpark.
	
Llevo unos meses viendo de lejos la posibilidad de utilizar C\# con Spark mediante el nuevo (para mí) proyecto de Microsoft
[.Net for Apache Spark](https://dotnet.microsoft.com/apps/data/spark) (a partir de ahora Spark.Net).
	
Hasta ahora parecía demasiado básico, pero tras ver los últimos vídeos y, sobre todo, la integración con Databricks que
parece tener buena pinta, me he decidido a experimentar un poco. No tengo problema en programar con Python o Scala, pero
me siento mucho más cómodo con C#, por tanto, para mí al menos, escribir aplicaciones en mi lenguage preferido es una
ventaja (siempre y cuando funcione, por supuesto).
	
Mi intención era seguir el [primer tutorial](https://docs.microsoft.com/es-es/dotnet/spark/tutorials/get-started),
un ejemplo que todo el que haya trabajado con PySpark conoce: contar las palabras de un archivo. Sencillo.

El tutorial está bien, excepto que ya tengo [instalado Spark en WSL](/blog/articles/big-data/instalacion-spark/instalacion-spark) y
por tanto las instrucciones de cómo instalar Spark.Net sobre Windows no me sirven, de hecho me han confundido más que ayudar, así
que comencemos por instalar Spark.Net en nuestro WSL.

## Instalación de .Net for Apache Spark sobre WSL

Supongo que los programadores de .Net en Linux ya habrán pasado por ésto, pero yo que soy nuevo en esto de WSL aún no había instalado
.Net Core en mi sistema. Por tanto lo primero es instalarlo. Las instrucciones de instalación son las mismas que la
[instalación de .Net Core en Ubuntu](https://docs.microsoft.com/en-gb/dotnet/core/install/linux-ubuntu),
por tanto, sigamos las instrucciones.
	
Lo primero que tenemos que hacer es añadir la clave de Microsoft a la lista de claves de confianza para el administrador de paquetes:

```bash
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
```

Y posteriormente, instalar el runtime de Net Core 3.1 (por supuesto, si queréis compilar utilizando WSL, deberéis instalar
el SDK no el runtime):

```bash
sudo apt-get update; \
sudo apt-get install -y apt-transport-https &amp;&amp; \
sudo apt-get update &amp;&amp; \
sudo apt-get install -y dotnet-runtime-3.1
```

En teoría eso es todo, si ejecutáis el comando `dotnet` debería aparecer ya la información sobre la versión de .Net Core.

En mi máquina, lo anterior no funcionó. No debo ser el único porque hay una sección de problemas con APT en esta misma página de la que obtuve
los siguientes comandos:

```bash
sudo dpkg --purge packages-microsoft-prod &amp;&amp; sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install dotnet-runtime-3.1
```

Afortunadamente, esta vez funcionó sin problema y no tuve que utilizar el párrafo que continuaba el artículo 
que parecía mucho más preocupante.

Así pues, ya tenemos .Net Core instalado en nuestro sistema WSL, lo siguiente es descargar la versión de Linux de .Net for Apache Spark.
	
Podemos conseguir la última versión de la librería en la sección de releases de 
[GitHub](https://github.com/dotnet/spark/releases), en concreto, podemos utilizar
los siguientes comandos para descargar, descomprimir y mover los archivos descomprimidos a una carpeta:

```bash
wget https://github.com/dotnet/spark/releases/download/v0.11.0/Microsoft.Spark.Worker.netcoreapp3.1.linux-x64-0.11.0.tar.gz
tar xvf Microsoft.Spark.Worker.netcoreapp3.1.linux-x64-0.11.0.tar.gz
sudo mv Microsoft.Spark.Worker-0.11.0/ /opt/SparkNetWorker
```

He dejado la librería sobre `/opt/SparkNetWorker` pero siéntase libre de moverlo a cualquier otro lugar.

Por último tenemos que añadir una variable de entorno (`DOTNET_WORKER_DIR`) con el directorio donde se encuentra .Net for Apache Spark. Para
eso editamos el archivo de configuración:
	
```bash
nano ~/.bashrc
```

Al final de ese archivo, añadimos la variable con la siguiente línea:
 
```bash
export DOTNET_WORKER_DIR = "/opt/SparkNetWorker"
```

Por supuesto, recuerde cambiar el nombre de directorio si ha dejado las librerías en otro lugar.

Por último sólo debemos refrescar la configuración:

```bash
source ~/.bashrc
```

En teoría, ya está todo instalado, sólo nos queda crear una aplicación que podamos ejecutar.

## Creación de una aplicación con Spark.Net

Para este ejemplo, he seguido los pasos del [tutorial](https://docs.microsoft.com/es-es/dotnet/spark/tutorials/get-started) que mencionaba al principio
del artículo, pero por si acaso no lo tiene a mano:
	
* Cree un nuevo proyecto de consola con .Net Core 3.1 (yo lo he llamado **TestSparkNet** pero de nuevo...)
* Añada el paquete Nuget `Microsoft.Spark`.
* Sobrescriba el contenido del archivo `Program.cs` con este código:

```csharp
using System;
using Microsoft.Spark.Sql;

namespace TestSparkNet
{
	class Program
	{
		static void Main(string[] args)
        {
            // Create a Spark session.
            SparkSession spark = SparkSession
                .Builder()
                .AppName("word_count_sample")
                .GetOrCreate();

            // Create initial DataFrame.
            DataFrame dataFrame = spark.Read().Text("input.txt");

            // Count words.
            DataFrame words = dataFrame
                .Select(Functions.Split(Functions.Col("value"), " ").Alias("words"))
                .Select(Functions.Explode(Functions.Col("words"))
                .Alias("word"))
                .GroupBy("word")
                .Count()
                .OrderBy(Functions.Col("count").Desc());

            // Show results.
            words.Show();

            // Stop Spark session.
            spark.Stop();
        }
	}
}
```

El código anterior lee un archivo llamado `input.txt`, así que una vez compilado, cree ese archivo en el directorio de depuración
e introduzca algunas líneas de texto.
	
Antes de continuar, echemos un vistazo a los archivos del proyecto:

![Archivos de proyecto](/blog/articles/big-data/experimento-spark-net/images/Archivos-proyecto.jpg "Archivos del proyecto de ejemplo")
			
Si nos fijamos, vemos que el paquete Nuget ha añadido dos paquetes jar a nuestro
proyecto: uno de ellos es importante en la ejecución de la aplicación que veremos en el siguiente paso.

## Último paso: ejecutar sobre Spark

Lo último que nos queda es compilar y ejecutar la aplicación en Spark. Para ello, nos colocamos en el directorio donde se ha
	compilado la aplicación y utilizamos el comando `spark-submit` de esta forma:
	
```bash
spark-submit \
	--class org.apache.spark.deploy.dotnet.DotnetRunner \
	--master local \
	microsoft-spark-2.4.x-0.11.0.jar \
	dotnet TestSparkNet.dll
```

Si os fijáis, `microsoft-spark-2.4.x-0.11.0.jar` es uno de los paquetes jar que nos ha instalado el paquete Nuget. Parece previsible
pensar que en el futuro esta versión cambie. Téngalo en cuenta si lee este artículo dentro de un par de meses.
	
Y tras unos segundos de incertidumbre con infinidad de líneas de log en la pantalla (Spark puede ser bastante dicharachero en algunos casos),
podemos ver el resultado de la aplicación:

![Resultado de la aplicación Spark.Net](/blog/articles/big-data/experimento-spark-net/images/Resultado-aplicacion.jpg "Resultados de la aplicación de ejemplo")
			
Sí, sé que no parece muy impresionante, pero para ser el primer experimento no está mal y al fin y al cabo parece demostrado que podemos
	ejecutar aplicaciones .Net sobre Spark en WSL sin complicarnos demasiado la vida.
	
Seguiré estudiando un poco más y preparando aplicaciones más complejas. Continuaremos informando.