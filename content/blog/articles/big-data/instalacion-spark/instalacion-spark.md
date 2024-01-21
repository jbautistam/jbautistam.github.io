+++
author = "Jose Antonio Bautista"
title = "Instalación de Spark en local"
date = "2020-10-10"
description = "Instalación de Spark en local con WSL"
thumbnail = "articles/big-data/experimento-spark-net/images/Experimento-Spark-Net.jpg"
tags = [ "spark", "wsl" ]
categories = [ "spark" ]
+++

En estos últimos meses, me he tenido que enfrentar con mayor o menor éxito con [Spark](https://spark.apache.org/).
	
A muy alto nivel, Spark es un motor de proceso distribuido al que podemos mandar trabajos utilizando Python, Scala, R y SQL.

Estos días he estado instalándolo sobre mi equipo con WSL y creo que es interesante detallar los pasos seguidos (sobre todo
para no olvidarme).
	
En primer lugar, aunque se puede instalar directamente sobre Windows, me interesaba configurarlo sobre WSL más que nada por
no 'ensuciar' mi sistema. Si algo sale mal, simplemente puedo desinstalar el sistema Linux sin tener que hacer nada más. Supongo
que alguien lo considerará una herejía pero en este caso he utilizado WSL más como contenedor que como herramienta de trabajo.
	
Lo primero que tenemos que hacer por tanto es instalar WSL, para ello vamos a **Configuración > Aplicaciones > 
Programas y características** y seleccionamos **Activar o desactivar las características de Windows*, donde debemos
activar, por supuesto, el **Subsistema de Windows para Linux** (que no sé vosotros, pero opino que debería
llamarse al revés).
	
![Activar o desactivar las características de Windows](/blog/articles/big-data/instalacion-spark/images/Caracteristicas-de-Windows.jpg "Activar o desactivar las características de Windows")
			
Una vez instalado (que si no me equivoco, necesita un reinicio)source-code/big-data/big-data/al Store de Windows y seleccionamos un sistema
Linux. Yo he elegido Ubuntu, más que nada por el instalador de paquetes, pero sentíos libres:

![Ubuntu](/blog/articles/big-data/instalacion-spark/images/Ubuntu.jpg "Ubuntu")
	
Cuando esté instalado, al ejecutarlo, nos pedirá el usuario y la clave del administrador y a partir de ese momento podemos trabajar
con el shell sin problemas.
	
Supongo que es cosa de costumbre, pero particularmente prefiero trabajar con [cmder](https://cmder.net/) abriendo una consola de tipo WSL en lugar de directamente
con la propia consola de Ubuntu, más que nada por los comandos de copiar / pegar y cosas por el estilo pero supongo que es cuestión
de gustos.
	
A partir de ahora, podemos ejecutar los comandos de instalación.

Primero vamos a actualizar tanto el instalador de paquetes como el sistema:

```bash
sudo apt update
sudo apt -y upgrade
```

Y ahora vamos a instalar Java (recordemos que Spark está escrito en Scala, por tanto necesitamos el JVM):

```bash
sudo apt install default-jre
sudo apt install openjdk-11-jre-headless
sudo apt install openjdk-8-jre-headless
```

Ahora ya podemos instalar Spark. Para ello vamos a seleccionar la versión desde la 
[página de descargas](https://spark.apache.org/downloads.html):
	
![Página de descargas de Spark](/blog/articles/big-data/instalacion-spark/images/Download-spark.jpg "Página de descargas de Spark")
			
Yo he elegido la versión 2.4.4 con Hadoop2.7 (spark-2.4.4-bin-hadoop2.7) así que vamos a descargarlo, descomprimirlo y dejarlo
en un directorio un poco más amigable:
	
```bash
wget https://www-us.apache.org/dist/spark/spark-2.4.4/spark-2.4.4-bin-hadoop2.7.tgz
tar xvf spark-2.4.4-bin-hadoop2.7.tgz
sudo mv spark-2.4.4-bin-hadoop2.7/ /opt/spark 
```

Por si alguien se lo pregunta, Spark no necesita Hadoop instalado para funcionar, pero sí que necesita las clases dsource-code/big-data/big-data/eso es lo que hace referencia el nombre de archivo. Si queréis utilizar Hadoop lo tenéis que instalar aparte y en ese
caso sí podríais descargar la versión de Spark sin Hadoop en el nombre.
	
Pues vamos a configurar Spark, para ello abrimos el archivo de configuración:

```bash
nano ~/.bashrc 
```

**Nota:** si tenéis instalado Visual Studio Code también podéis ejecutar code ~/.bashrc

Al final de este archivo debemos insertar estas líneas:

```bash
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
```

Las dos primeras configuran el directorio de Spark y lo añaden al entorno. La última configura la versión de Java, parece ser
que Spark 2.4 no se lleva bien con Java 11. 
	
Sólo nos queda refrescar la configuración:

```bash
source ~/.bashrc
```

Y estamos ya preparados. Spark ya está instalado, sólo nos queda ponerlo en marcha.

Primero vamos a arrancar un nodo master:

```bash
start-master.sh
```

A partir de este momento podemos abrir un navegador para ver la interface de usuario sobre `localhost:8080`:

![Interface de usuario de master](/blog/articles/big-data/instalacion-spark/images/Spark-UI-master.jpg "Interface de usuario de master")
			
Abrir el navegador (al menos la primera vez), es importante porque nos va a dar la dirección del nodo master que vamos
a necesitar a la hora de añadir nodos al cluster. En mi caso: `spark://jbautistam-PC.localdomain:7077` que podéis ver
en la cabecera en negrita.
	
Ahora ya podemos añadir un nodo de trabajo:

```bash
start-slave.sh spark://dirección del nodo master:7077
```

Por supuesto, este es un nodo que corre sobre mi máquina pero para añadir nodos en otras máquinas el proceso es el mismo.

Y ya podemos ver el nodo worker en la UI:

![Interface de usuario con un nodo master y otro worker](/blog/articles/big-data/instalacion-spark/images/Spark-UI-Worker.jpg "Interface de usuario con un nodo master y otro worker")

Para el futuro, cuando queramos detener los nodos utilizaremos estas instrucciones:

```bash
$SPARK_HOME/sbin/stop-slave.sh
$SPARK_HOME/sbin/stop-master.sh
```

Un par de cosas más, podéis abrir la consola de Spark con:

```bash
spark-shell
```

![Shell de Spark](/blog/articles/big-data/instalacion-spark/images/Spark-shell.jpg "Shell de Spark")
			
**Nota:** para salir, escribid `:quit` (de nada).

Esta es la consola predefinida, con Scala, pero si sois más de Python, podéis instalar PySpark utilizando pip:

```bash
sudo apt install python-pip
pip install pyspark
```

**Nota:** la primera instrucción es precisamente para instalar pip.

Para ejecutar la consola de Python, simplemente ejecutad `pyspark`:

![Pyspark](/blog/articles/big-data/instalacion-spark/images/Pyspark.jpg "Pyspark")
			
Por cierto, tenéis un directorio `examples` con ejemplos de aplicaciones en Scala, Python y R y algunos archivos CSV, parquet, 
Avro y json para pruebas.

## Conectando a Spark con JDBC

Como hemos dicho, podemos acceder a Spark utilizando Psource-code/big-data/big-data/ y R, pero si sois como yo habréis escuchado hablar de
Spark Sql, por eso empezó todo precisamente.
	
El caso es que la documentación de Spark es muy buena y puedes utilizar sin problemas SQL desde Python pero no hay una forma
sencilla de ejecutar sentencias SQL sin tener que definir dataframes, RDD, UDF y todas esas cosas extrañas que le gustan a la
gente que trabaja con datos.
	
No dudo de su utilidad, pero yo lo único que quiero es conectame a Spark y lanzarle sentencias SQL y para ello tengo que dar un
par de pasos más.
	
Concretamente, tenemos que lanzar el servidor JDBC / ODBC ejecutando la siguiente instrucción:

```bash
$SPARK_HOME/sbin/start-thriftserver.sh
```

Un comentario sobre ésto, el resultado de este comando es algo así:

```bash
starting org.apache.spark.sql.hive.thriftserver.HiveThriftServer2, 
logging to /opt/spark/logs/spark-jbautistam-org.apache.spark.sql.hive.thriftserver.HiveThriftServer2-1-jbautistam-PC.out
```

No da ningún error aunque no funcione (al menos a mí no me lo ha dado), los errores los encontraréis en el archivo de log. Tardé
un par de horas en encontrar que no levantaba el servicio porque la versión de Java estaba mal definida.
	
El caso es que si todo ha ido bien, ya podéis entrar en la interface de usuario de Thrift en `localhost:4040`:

![Interface de usuario Thrift](/blog/articles/big-data/instalacion-spark/images/Thrift-UI.jpg "Interface de usuario Thrift")
			
Que parece muy interesante pero en realidad no nos permite lanzar ninguna consulta, sólo ver el estado de los trabajos (y creedme,
la consultaréis a menudo).
	
Siempre podemos utilizar la consola con `spark-sql`: 

![Spark Sql](/blog/articles/big-data/instalacion-spark/images/Spark-sql.jpg "Spark Sql")
		
Que francamente, acostumbrado al manager de SQL Server, tampoco me entusiasma. Pero ya que tenemos un servidor JDBC vamos a abrir 
[DBeaver](https://dbeaver.io/) contra él. 
	
Si no lo habéis utilizado nunca, Dbeaver es un aplicación para administrar todo tipo de base de datos: Postgres, 
MySql, Sql Server, SqLite y sobre todo (y por lo que nos interesa) Spark.
	
Así que abrimos Dbeaver y creamos una conexión a Spark:

![Conexión a Spark con Dbeaver](/blog/articles/big-data/instalacion-spark/images/Conexion-dbeaver.jpg "Conexión a Spark con Dbeaver")
			
y la configuramos:

![Configuracion de la conexión con Dbeaver](/blog/articles/big-data/instalacion-spark/images/Configuracion-dbeaver.jpg "Configuracion de la conexión con Dbeaver")

En este caso, que voy a conectarme a mi equipo, no me hace falta usuario y contraseña pero por supuesto, se puede configurar
en el servidor. 
	
Cuando pulsemos sobre **Aceptar**, la primera vez se descargará los drivers JDBC adecuados y nos abrirá la conexión a
nuestro servidor:

![Dbeaver conectado a Spark](/blog/articles/big-data/instalacion-spark/images/Dbeaver-con-Spark.jpg "Dbeaver conectado a Spark")
			
Ahora, ya podemos lanzar consultas SQL contra nuestro servidor Spark, por ejemplo, vamos a consultar con SQL uno de esos 
archivos parquet de ejemplo de los que hablábamos hace un rato:
	
```sql
SELECT * 
	FROM parquet.`examples/src/main/resources/users.parquet`
```

y vemos el resultado:

![Resultado consulta con Dbeaver](/blog/articles/big-data/instalacion-spark/images/dbeaver-query.jpg "Resultado consulta con Dbeaver")
		
También podemos ir a la consola de Thrift para ver cómo se ha ejecutado todo:

![Información sobre procesos de Thrift](/blog/articles/big-data/instalacion-spark/images/Thrift-jobs.jpg "Información sobre procesos de Thrift")
	
**Nota:** la documentación de Spark Sql en la propia web de Spark es bastante reducida, 
simplemente nos redirigen a la de Hive pero para mi gusto la de 
[Databricks](https://docs.databricks.com/spark/latest/spark-sql/index.html#sql-language-manual)
está bastante más cuidada. 
	
Pues eso es todo, en poco más de 20 minutos tenemos un entorno de Spark montado al que podemos acceder no sólo con
Python y Scala si no también con SQL.