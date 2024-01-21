+++
author = "Jose Antonio Bautista"
title = "Cosas que me gustaría que me hubiesen explicado sobre databricks"
date = "2020-10-10"
description = "Algunas cosas que me gustaría que alguien me hubiese explicado cuando empecé a desarrollar con Databricks"
thumbnail = "articles/big-data/cositas-databricks/images/Cositas-Databricks.jpg"
featured = true
tags = [ "spark", "databricks", "featured" ]
categories = [ "spark" ]
+++

Llevo algunos meses trabajando con [DataBricks](https://databricks.com/) en local y me he encontrado algunas rarezas que he tenido que ir puliendo poco a poco.

Me habría ahorrado un montón de dolores de cabeza si alguien me las hubiese contado previamente, por eso escribo este
artículo. Por si ayuda a alguien en una situación parecida. 

En primer lugar me gustaría explicar nuestro método de desarrollo con **DataBricks**. 

Si no lo has utilizado, simplemente comentar que, a grandes rasgos, Databricks es una herramienta montada sobre 
Spark que facilita el trabajo (y mucho) a la hora de creación y mantenimiento de clusters y manejo de notebooks.
	
Databricks está muy orientado a entornos de ciencia de datos utilizando notebooks en Python, SQL, Scala o R. 

No tengo nada en contra de programar a base de notebooks pero cuando llevas un par de horas con ellos comienzas a echar de menos
un editor como dios manda. No sólo por la depuración si no también porque los scroll hacen cosas, por ser suaves, raras, la edición
a base de celdas es, digámoslo así, ligeramente complicada, la búsqueda tiene detalles en los que preferiría no ahondar, la
integración con el control de versiones y las ramas es algo peculiar, etc, etc. 
	
Además tiene un segundo problema mucho más grave: el coste. Depurar contra clusters cuando estamos escribiendo una aplicación 
que probamos paso a paso veinte veces al día puede llegar a resultar bastante caro.
	
Nuestra solución pasa por programar directamente en [Spark Sql en local](/blog/articles/big-data/instalacion-spark/instalacion-spark)
utilizando una aplicación que desarrollamos nosotros: [BauDbStudio](Aplicaciones\BauDbStudio\Manual).

**Nota:** El manual no está completo aún y la versión en la Web está algo desactualizada, prometo
ponerme las pilas con ello pero últimamente el tiempo me puede.
	
Por concretar: escribimos scripts en Spark Sql en BauDbStudio, los lanzamos con ODBC contra Spark en local mientras que dearrollamos
y probamos. Cuando terminamos, le pasamos un proceso que transforma estos scripts a notebooks que entienda DataBricks, 
lo subimos a un repositorio especial y utilizamos Azure DevOps para desplegar sobre DataBricks. 
	
Sencillo ¿no?
	
¿Por qué Spark Sql y no Python? Me gusta Python, de hecho, algunos módulos de nuestros procesos como el lanzador de
notebooks de SQL están escritos en Python (no ha habido otro remedio, por cierto) pero para tratar datos, creo que SQL
es bastante más natural.
	
Y si todo funciona ¿qué puede ir mal?

Como comentaba, DataBricks está basado en Spark pero a la hora de desplegar nuestros scripts a notebooks nos encontramos con
algunos problemas.
	
La primera vez que subimos un notebook desde nuestro repositorio, nos dimos cuenta que, al abrirlo en DataBricks nos escribía un 
punto en rojo delante del primer texto. De hecho, se negaba a ejecutarlo a menos que borrásemos a mano ese punto rojo
que identificaba como un carácter extraño.

¿Por qué? Bueno, esa fue la primera cosa que nadie me explicó.

En principio parecía un problema de codificación: probé con UTF-8 (el correcto), con ASCII, con UTF-16 y obtuve diferentes errores
y muchos acentos y años escritos raro (sí, mis comentarios están en español ¿algún problema?).
	
El caso es que el error no es de la codificación aunque sí tiene que ver con la codificación: cuando se escribe un archivo UTF-8, los
primeros caracteres identifican el archivo. Es lo que se llama [BOM](https://es.wikipedia.org/wiki/Marca_de_orden_de_bytes).
Databricks no reconoce estos caracteres en sus notebooks así que para que funcione, nuestros archivos se deben escribir en UTF-8 pero
sin los caracteres de BOM.
	
¿Cómo me di cuenta? Descargando un notebook válido de Databricks y comparándolo con los míos utilizando un editor hexadecimal. 
Sí, como en los viejos tiempos.
	
Otro problema: en Spark SQL para leer un archivo parquet utilizamos sentencias de este estilo:

```sql
SELECT *
	FROM parquet.`/mnt/raw/CustomerOrders.parquet`
```

Probándolo en local (en mi máquina con WSL sobre Windows) funciona perfectamente. Al ejecutarlo en Databricks nos daba un error similar a 
"no existe el archivo CustomerOrders.parquet"
	
Pero en mi storage está el archivo `customerOrders.parquet` ¿cuál puede ser el problema?

El lector avispado ya se habrá dado cuenta: Unix. 

Unix es sensible a mayúsculas, por tanto debemos poner los nombres de archivos
exactamente igual a como estén en el storage. Y más vale que no te equivoques. Nunca. Te puedo asegurar que si ejecutas un proceso 
en un cluster de ocho nodos durante dos horas y falla en la última lectura de un archivo porque no está correctamente escrito no 
te hace ninguna gracia. Le ha pasado a un amigo.
	
Dado que es bastante difícil que nos acordemos de escribir correctamente todos los nombres de archivos (en mi caso, unos doscientos),
tomamos una decisión salomónica: en el storage todos los nombres de archivos se escriben en minúsculas y modificamos el proceso de generación
de los notebooks de Databricks para que buscara en nuestros scripts todo lo que estuviera entre acentos y lo pusiera en minúsculas.
Esta fue fácil.
	
El siguiente problema vino con los argumentos y también está relacionado con las mayúsculas y minúsculas. En Spark Sql puedo escribir
algo así:

```sql
SELECT *
	FROM parquet.`/mnt/raw/CustomerOrders.parquet`
	WHERE Customer = $CustomerId
```

Donde `$CustomerId` es un parámetro de la consulta.

Si tenemos este script en un notebook y lo queremos ejecutar desde otro escrito por ejemplo en Python, 
lo que tenemos que hacer es pasarle este argumento en la instrucción `run` de `dbutils`:
	
```python
# Crea el diccionario de parámetros de ejecución
parameters = { 
				"customerId": 28
			}

# Ejecuta el script de SQL
dbutils.notebook.run("./Script.sql", 3600, parameters)
```

En este caso, como decíamos antes, Databricks también es sensible a mayúsculas y la ejecución de la sentencia SQL nos da un error
similar a *no puedo encontrar el parámetro CustomerId*. Por tanto: siempre que utilicemos un argumento tiene que estar capitalizado
exactamente igual que en el diccionario de parámetros.
	
Por supuesto, también nos resulta bastante difícil obligar a todo el equipo a que escriba los parámetros con el mismo nombre constantemente
más cuando en local, Spark Sql no nos da ningún error (y particularmente no me apetecía escribir un intérprete de SQL para éso). 
Por eso, al convertir los scripts a notebooks otra de las cosas que hacemos es cambiar los
nombres de los argumentos a minúsculas. En el diccionario en Python están también escritos en minúsculas, por supuesto.
	
Una de las cosas que me costó más trabajo fue precisamente entender cómo trabaja Databricks con los argumentos de SQL. En concreto
tenía una sentencia SQL como ésta:

```sql
SELECT Name + $separator + LastName 
	FROM parquet.`/mnt/raw/CustomerOrders.parquet`
	WHERE Customer = $CustomerId
```

Curiosamente cuando le pasaba el argumento `separator` con un punto, por ejemplo, me daba un error y me indicaba la SQL que estaba
intentando ejecutar. Concretamente ésta:

```sql
SELECT Name + . + LastName 
	FROM parquet.`/mnt/raw/CustomerOrders.parquet`
	WHERE Customer = $CustomerId
```

¿Qué ha pasado? ¿Por qué razón ha sustituido directamente el argumento en la SQL sin ponerle apóstrofes alrededor? Gran misterio
pero así fue.
	
El caso es que la documentación te dice que no utilices `$argumento` si no la función `getargument` dado que la primera forma
va a dejar de funcionar en breve. Es decir, la sentencia anterior debería ser:

```sql
SELECT Name + getArgument("separator") + LastName 
	FROM parquet.`/mnt/raw/CustomerOrders.parquet`
	WHERE Customer = getArgument("CustomerId")
```

Pero teníamos el mismo problema que antes: en Spark Sql funciona correctamente así que, por comodidad, decidimos dejar todos nuestros 
scripts de SQL utilizando argumentos a la forma antigua (con un carácter dólar por delante) y modificar el proceso de generación de
notebooks para que sustituyese las cadenas `$parameter` por `getArgument("parameter")` (por supuesto, en minúsculas todo).

Por cierto, tengo que avisarte que he hecho trampa. Cuando escribía ésto:

```python
# Ejecuta el script de SQL
dbutils.notebook.run("./Script.sql", 3600, parameters)
```

En realidad no funciona. Databricks se niega a ejecutarlo. Dice que no existe ese notebook. Pero existe, lo estoy viendo. Y
está bien escrito, no me vas a pillar otra vez con lo de mayúsculas y minúsculas.
	
La verdad es que para que funcione tienes que escribir ésto:

```python
# Ejecuta el script de SQL
dbutils.notebook.run("./Script", 3600, parameters)
```

Es decir: cuando ejecutas un notebook con extensión SQL no puedes poner la extensión **.sql** en la intrucción `run. Si lo
pones te va a dar un error.

No ocurre lo mismo si el notebook está escrito en Python y tiene extensión **.py**. En ese caso, tienes que escribir la extensión
en la instrucción `run`. Si no te va a dar un error de que no encuentra el archivo.
	
¿Por qué funciona diferente? ... ¿Realmente me lo estás preguntando a mí?

Una cosa que me resultó curiosa. Si ejecutas ésto en Spark Sql no hay ningún problema:

```sql
DROP TABLE IF EXISTS Customers
```

Pero en Databricks da una excepción si no existe. ¿Cómo? ¿Una excepción? Sí, no me mires así, a mí también me sorprendió.

Cualquier servidor de base de datos que se precie, cuando ejecuta esa instrucción comprueba si existe la tabla y si existe
la elimina, si no existe continúa con la siguiente instrucción. 
	
Bueno, pues Databricks no, Databricks da un error. Bueno,
realmente da un error de vez en cuando. Veamos la documentación de la instrucción 
[DROP TABLE](https://docs.databricks.com/spark/latest/spark-sql/language-manual/drop-table.html):
	
![DROP TABLE en Databricks](/blog/articles/big-data/cositas-databricks/images/Error-drop-table.jpg "Instrucción DROP TABLE en Databricks")

Lo dice bien claro: si la tabla no existe, lanza una excepción. A veces.

Os cuento. En la práctica puedes utilizar bases de datos en Databricks (para mí no son bases
de datos si no esquemas pero para el caso es lo mismo). En caso que utilices un nombre de base de datos creado
previamente (no la base de datos **default** que se utiliza cuando no le ponemos nombre de base de datos), 
la instrucción funciona sin problemas:

```sql
DROP TABLE IF EXISTS DbOrders.Customers
```

Y una última cosa: esta es difícil y posiblemente me equivoque pero vamos a hablar de niveles de aislamiento.

Databricks nos permite ejecutar trabajos sobre un cluster. En nuestro caso, lanzo esos trabajos utilizando la librería de
Azure Databricks pero es exactamente igual cuando utilizamos la API REST. El caso es que cada trabajo se ejecuta en
un cluster y cada trabajo debe ejecutarse contra un blob storage diferente porque cada uno va con datos diferentes,
es decir, un multitenant de libro.
	
La teoría es que estos cluster deben poder ejecutarse a la vez y en teoría deberían ser aislados pero ¿cómo ejecuta
	databricks estos cluster? ¿cuál es su nivel de aislamiento? Bueno, pues digamos simplemente que 
	[depende](https://docs.databricks.com/notebooks/notebooks-use.html):
	
![Databricks Isolation levels](/blog/articles/big-data/cositas-databricks/images/Isolation-levels.jpg "Isolation levels")

¿Os ha quedado claro? A mí no.

Hagamos una prueba. Vamos a montar un storage sobre un directorio de DBFS utilizando Python:

```python
# Obtiene los parámetros
containerName = getArgument("containername")
storageAccountName = getArgument("storageaccountname")
mountPoint = "/mnt/engine/raw"

# Crea la cadena de configuración
confKey= "fs.azure.account.key." + storageAccountName + ".blob.core.windows.net"

# Monta la unidad
try:
  dbutils.fs.mount(
                    source = "wasbs://" + containerName + "@" + storageAccountName + ".blob.core.windows.net",
                    mount_point = mountPoint,
                    extra_configs = { confKey : dbutils.secrets.get(scope = "KeyVaultScope", key = "blob-storage-access-key") }
                  )  
except:
  print("Cant mount the storage. Continue")
```

Más o menos como dicen todos los ejemplos. Lo único reseñable es que utilizamos nuestro amigo `getargument` para recoger los
argumentos del job.
	
Ahora bien ¿qué ocurre cuando ejecutamos dos clusters a la vez con ese código?

Pues que el segundo que ejecute nos va a dar un error diciendo que no puede montar la unidad. ¿Por qué si el nombre de contenedor
es diferente? Pues creo que porque el nombre del "punto de montaje" (variable `mountPoint`) es igual en ambos casos.
	
Reconozco que en este caso aún tengo mis dudas. En todas mis pruebas me ha dado el mismo error
pero me extraña tanto que me pregunto si algo estaré haciendo mal. Así que si alguien sabe porqué ocurre ésto por favor que
no dude en corregirme.
	
¿Y la solución? En mi caso: cambiar el contenido de la variable `mountPoint` para que fuera diferente en cada ejecución
(en este caso incluyendo el nombre de contenedor).

```python
# Obtiene los parámetros
containerName = getArgument("containername")
storageAccountName = getArgument("storageaccountname")
mountPoint = "/mnt/" + containerName + "/raw"

# .... el resto del código es igual al ejemplo anterior ...
```

Digamos simplemente que en mi cluster funciona... por ahora.

Vamos a dar un pasito más...

Como decíamos antes, en Spark Sql tenemos una instrucción para crear una base de datos, de hecho, si no creamos una base de datos
los `DROP TABLE` se van a comportar de una forma extraña, por eso, mejor montar una base de datos. Lo vamos a hacer así:
	
```sql
CREATE DATABASE IF NOT EXISTS DbCustomer
	LOCATION '/mnt/engine/raw/DbCustomer'
```

¿Veis el problema? Mirad de nuevo.

El problema es la ubicación: `/mnt/engine/raw/DbCustomer`. Antes no fallaba pero dado que hemos cambiado en Python el
nombre del directorio de montaje por `"/mnt/" + containerName + "/raw"`, la creación de base de datos no va a funcionar en la
vida.
	
Tenemos un segundo problema (o duda): ¿cuál es el nivel de aislamiento de bases de datos en los clusters? ¿cuando ejecuto
dos cluster los dos con el mismo nombre de base de datos entienden que son bases de datos separadas o la consideran
global para todos ellos? Es decir, si yo ejecuto ésto:
	
```sql
SELECT *
	FROM DbCustomer.Orders
```

en dos clusters al mismo tiempo ¿van a la misma base de datos / esquema o entienden que son diferentes?

Pues he buscado en Google y no he encontrado documentación al respecto. Parece que la documentación de Spark no dice nada 
y la de Databricks menos. De nuevo: si alguien sabe qué ocurre estoy ansioso de aprender.
	
El caso es que, por mis pruebas, parece que si dos cluster se ejecutan a la vez con el mismo nombre de base de datos, la comparten.
Por supuesto, no es lo que quiero.
	
Pero eso me supone un problema: ¿cómo puedo escribir un script en SQL que se ejecute para diferentes clientes con nombres diferentes
de base de datos?
	
Vayamos por partes. En primer lugar vamos a solucionar el problema del `LOCATION` en la instrucción `CREATE TABLE`.

Para ello, tenemos que enviar a nuestra SQL el valor de la variable `mountPoint`. Más o menos así:

```python
# Crea el diccionario de parámetros de ejecución
parameters = { 
	"customerId": 28,
	"mountpoint": mountPoint
	}
			
# Ejecuta el script de SQL
dbutils.notebook.run("./Script", 3600, parameters)
```

y vamos a cambiar nuestra sentencia de creación de base de datos por ésto:

```sql
CREATE DATABASE IF NOT EXISTS DbCustomer
	LOCATION '$mountpoint/DbCustomer'
```

¿Habéis visto? Ponemos `mountpoint` como argumento y nos aprovechamos de la forma de trabajar de Databricks con los parámetros 
que al principio nos dio tantos quebraderos de cabeza. Dado que se va a sustituir `mountpoint` por la cadena sin poner 
apóstrofes alrededor, nuestra sentencia se va a traducir en ésto:
	
```sql
CREATE DATABASE IF NOT EXISTS DbCustomer
	LOCATION '/mnt/nombrecontenedor/raw/DbCustomer'
```

Me gusta esta forma de aprovechar los errores a nuestro favor aunque habrá que ver qué pasa cuando los argumentos ya no puedan
escribirse con un dólar por delante y sea obligatorio utilizar `getargument` ¿seguirá funcionando?

De cualquier forma, tenemos el mismo problema con el nombre de base de datos así que
vamos a hacer algo similar, le vamos a pasar un argumento más a nuestro script de SQL con el nombre de base de datos:

```python
# Creamos un nombre único de base de datos con el nombre del contenedor
dbName = "DbCustomer" + containerName
# Crea el diccionario de parámetros de ejecución
parameters = { 
	"customerId": 28,
	"mountpoint": mountPoint,
	"dbname", dbName
}

# Ejecuta el script de SQL
dbutils.notebook.run("./Script", 3600, parameters)
```

Y ahora sólo tenemos que utilizar ese argumento en todas nuestras consultas (incluyendo la creación):

```sql
CREATE DATABASE IF NOT EXISTS $dbname
	LOCATION '$mountpoint/$dbname'
	
SELECT *
	FROM $dbname.Orders
```

Perfecto, ahora funciona correcta e independientemente en los clusters. 

Aún así, tenemos un problema: ¿qué pasa en local? En Spark Sql los nombres de argumentos
utilizan, digámoslo así, el método SQL. Por tanto, me pondrá apóstrofes alrededor del valor del argumento y me estropeará todo mi código.
	
En realidad, dado que BauDbStudio lo desarrollamos nosotros, no nos cuesta demasiado "trucar" un poco el código de ejecución.
Esa es una ventaja de desarrollar nuestras propias herramientas (hay muchas desventajas pero por ahora centrémonos en lo bueno).

Digamos simplemente que en nuestro código en local tenemos estas sentencias:

```sql
CREATE DATABASE IF NOT EXISTS {{DbName}}
	LOCATION '{{MountPoint}}/{{DbName}}'
	
SELECT *
	FROM {{DbName}}.Orders
WHERE CustomerId = $CustomerId
```

Y tenemos un archivo de parámetros de este estilo:
	
```json
{
	"Constant.MountPath": "/mnt/c/Data",
	"Constant.DbCompute": "DbCompute_Cust",
	"CustomerId": "PER_20"
}
```

Cuando se ejecuta desde BauDbStudio, se sustituyen las cadenas de tipo `{{argumento}}` por los parámetros identificados como 
constantes en el JSON (en este caso `Constant.argumento`) y los argumentos como `CustomerId` por los parámetros con
el mismo nombre en el archivo JSON (en este caso `CustomerId`).
	
Nuestro proceso de conversión a notebooks hace algo similar, sustituye las constantes por el nombre del argumento que se
le va a pasar en el archivo Python y los parámetros por llamadas a `getArgument` como hemos visto antes.
	
Eso nos permite tanto ejecutar en local como en los clusters sin hacer modificaciones a nuestro código.

**Nota:** Por supuesto, esto debería estar explicado en el manual de BauDbStudio. Un poco de paciencia, por favor.
	
## Conclusiones

Para terminar: Databricks es una gran herramienta pero como toda gran herramienta tiene sus cosas.

Este es nuestro modo de trabajo, así hemos solucionado los errores que nos hemos ido encontrando a lo largo de estos dos últimos
meses creando un entorno local para nuestros desarrollos en Spark Sql y ejecutando con clusters y notebooks de Databricks.
Espero que sirva de ayuda a quienes se encuentre con problemas similares. 

Como decía en varios puntos del artículo, si alguien tiene mejores soluciones o me he equivocado en algo, por favor, estoy más
que dispuesto a corregir este texto con sus comentarios.
	
Y por supuesto, si alguien quiere más información, sólo tiene que pedirla.
