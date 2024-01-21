+++
author = "Jose Antonio Bautista"
title = "Intérprete de scripts SQL"
date = "2020-10-10"
description = "Primeras iteraciones para crear un interprete de scripts en BauDbStudio para SQL"
thumbnail = "/articles/big-data/interprete-scripts-spark/Interprete-scripts-Spark.jpg"
tags = [ "spark", "wsl", "sql", "baudbstudio" ]
categories = [ "spark" ]
+++

Siempre he defendido SQL como un lenguaje ideal para el tratamiento de datos con una salvedad: la falta de instrucciones
imperativas en el estándar.
	
SQL es un lenguaje declarativo, de ahí viene realmente su potencia: ejecutar una instrucción indicando lo que deseo obtener no
cómo se va a obtener. 
	
Funciona aunque a los programadores, acostumbrados a lenguajes imperativos, se nos suele dar bastante mal cuando nos quitan `for`,
`if` y similares. Una vez te das cuenta que para trabajar con SQL debes olvidar los bucles y pensar en bloques de datos, todo va rodado.
	
Desgraciadamente, en ocasiones necesitamos instrucciones como `if`, `while`, declaraciones de variables, etc...
Hay formas de hacerlo utilizando SQL puro pero suelen ser bastante complicadas, por ejemplo, ¿quién sabría
decirme qué hace esta instrucción escrita en Spark Sql?
	
```sql
SELECT date_add(Date(CONCAT(CAST(Year(Current_Date()) - 5 AS string), '-01-01')), Row_Number() OVER (ORDER BY Counter) - 1) AS Date
   FROM (SELECT 1 AS Counter UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1) AS tmp1
   CROSS JOIN (SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1) AS tmp2
   CROSS JOIN (SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1) AS tmp3
   CROSS JOIN (SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1) AS tmp4
   CROSS JOIN (SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1) AS tmp5
```

Aunque no lo parezca, es sencillo: está obteniendo los días entre el uno de enero de hace cinco años hasta una fecha indeterminada (tendría
que ponerme a contar, lo siento). Es decir, esa serie de `SELECT / CROSS JOIN / UNION ALL`, junto con la función de ventana 
`Row_Number` sirven para definir un contador de 1 a n (de nuevo, me tendría que poner a contar).
	
Pero que se pueda, no implica que sea fácil, por eso la mayoría de los gestores de bases de datos relacionales avanzados incorporan
un lenguaje imperativo: SQL Server tiene TSql, Oracle tiene su PL/SQL, etc... 
	
Spark lo soluciona de una forma diferente, además de escribir sentencias en Spark Sql, podemos embeber Spark Sql en scripts de Python, la
combinación perfecta. Siempre que sepas Python, claro. No es nuevo, es lo que hacemos en la mayoría de los lenguajes cuando lanzamos
una sentencia SQL contra una conexión a base de datos.
	
Pero dejémonos de introducciones ¿a qué viene todo ésto? Pues todo esto viene a que mientras programo 
[BauDbStudio](/blog/applications/baudbstudio/manual/001-tutorial-baudbstudio/001-tutorial-baudbstudio), cuánto más ejecuto contra Spark Sql, más me
doy cuenta que necesito un lenguaje imperativo. Echo de menos los `if`, `for`, `while` y demás. No siempre, pero cuando
hace falta, tengo que reconocer que cuesta horrores pasarlo por alto.
	
Podría haber elegido la opción de ejecutar Python en **BauDbStudio**, hay librerías para hacerlo pero ¿qué ocurriría con 
otros gestores de bases de datos como SqLite o Postgress?, no me acababa de convencer (aunque no descarto acabar haciéndolo).
	
La otra forma era definir mi propio lenguaje imperativo e interpretarlo desde **BauDbStudio**. Esa fue la opción que elegí, por cacharrear, ya sabéis.
	
Aún está en fase de pruebas, pero ya podemos escribir scripts como éste:

```
declare int valor = 20;

if valor > 10
{
	<%
		DROP TABLE IF EXISTS {{DbCompute}}.TestScript
		GO
		
		CREATE TABLE {{DbCompute}}.TestScript AS
			SELECT *
				FROM {{DbCompute}}.Ventas
				WHERE Cantidad > #valor
		GO
	%>
}	
```

Como se puede ver, es una mezcla entre los [scripts de SQL](/blog/applications/baudbstudio/manual/040-scripts-sql/040-scripts-sql) que teníamos
hasta ahora en **BauDbStudio** con instrucciones SQL separadas por comandos `GO`.

Por el momento el lenguaje permite lo básico:

* Declaración de variables tipadas
* Cálculo de expresiones
* Definición de funciones
* Instrucciones condicionales (if, else)
* Instrucciones iterativas (for, while, do... while)
* Llamada a funciones
* Declaración de funciones predefinidas, es decir, aquellas que va a ejecutar el programa principal, no el intérprete (por ejemplo, print, dateAdd, ...)
* Ejecución de sentencias SQL
	
Suficiente para tener un lenguaje iterativo con SQL embebido para, en teoría, cualquier gestor de base de datos integrado en
**BauDbStudio** (en este momento, SqLite, SQL Server, PostgreSql, MySql y, por supuesto, Spark).
	
Estoy en fase de pruebas y moderadamente esperanzado. En un mes (modificando otros intérpretes que había desarrollado en el pasado), el
código ha avanzado mucho, ejecuta correctamente aunque le falta pulir algunos detalles como el paso de parámetros y cosas por
el estilo que aún dan algo de guerra.
	
Digamos que por ahora, el lenguaje ha quedado un poco *primitivo*, tenemos sentencias `declare` o `let` o `call` en lugar de inferir 
que estamos en una declaración de tipos o una variable o una llamada a función (ahora interpreta el código a partir de palabras reservadas).
También obliga a que las sentencias iterativas y condicionales (`if`, `while`, ...) estén dentro de un bloque de sentencias (con llaves) aunque
sólo tengan una sentencia. Supongo que el lenguaje evolucionará, pero por ahora era la forma más sencilla.

## ¿Dónde nos deja esto con **DataBricks**?

Actualmente puedo ejecutar en local las instrucciones de mi programa sin problema pero esto jamás va a poder subir a **Databricks**. Es
imposible que mi nuevo lenguaje se ejecute en ese entorno.
	
Podríamos hacer una cosa: reproducir el intérprete en Python de forma que recogiera el script y lo interpretara en tiempo de ejecución en
Databricks. Por supuesto no lo voy a hacer, entre otras cosas porque Python no es mi fuerte. También lo podríamos hacer utilizando
[Spark.Net](/blog/articles/big-data/experimento-spark-net/experimento-spark-net) con C# pero no creo que por hoy por hoy tenga los conocimientos
suficientes como para meterme en ese sembrao. 
	
La segunda opción (que me parece la mejor) es transpilar. 

Como sabemos, la primera fase de la interpretación es el análisis sintáctico que devuelve una colección de sentencias y 
expresiones en un lenguaje interno. La idea es transpilar este grupo de sentencias a un notebook de Python para databricks
con Spark Sql inmerso. Algo así:
	
```Python
valor = 20

if valor > 10:
	spark.Sql("DROP TABLE IF EXISTS DbCompute.TestScript")
	spark.Sql('''CREATE TABLE {{DbCompute}}.TestScript AS
					SELECT *
					FROM {{DbCompute}}.Ventas
					WHERE Cantidad > {}
			  ''', 
			  valor)
```

Ni que decir tiene que eso todavía ni lo he empezado pero la teoría es que debería funcionar.

## Pruebas unitarias

Y una pregunta lanzada al viento ¿cómo se crean pruebas unitarias para un intérprete?

La teoría es que a partir de un script de entrada debemos obtener una lista de sentencias que vienen del analizador léxico y sintáctico.
Podemos comprobar que esas sentencias sean las que esperamos, pero así estaríamos comprobando los analizadores, no el intérprete.
	
¿Cómo probamos entonces que la ejecución de un programa interpretado pasa por todos los puntos y obtiene los resultados correctos?

Pues no me parece fácil. En mi caso lo he solucionado aprovechando la forma en que está concebido el intérprete.

El intérprete en realidad no sabe nada de SQL. Por dos razones: en primer lugar porque el SQL de cada gestor de base de datos es
ligeramente diferente (los estándares, ya sabéis), por tanto escribir un intérprete común es prácticamente imposible. 
El intérprete sólo reconocer cadenas de comandos SQL (las que van entre `<%` y `%`>) y lanza una petición al procesador principal.
	
Lo mismo ocurre con las funciones intrínsecas como `print`, simplemente lanza una llamada al procesador.

Por tanto, en este código:

```
declare int valor = 20;

if valor > 10
{
	<%
		DROP TABLE IF EXISTS {{DbCompute}}.TestScript
		GO
		
		CREATE TABLE {{DbCompute}}.TestScript AS
			SELECT *
				FROM {{DbCompute}}.Ventas
				WHERE Cantidad > #valor
		GO
	%>
}	

call print("valor: " + valor);
```

Se va a llamar al procesador principal dos veces: la primera con el comando SQL, la segunda con un mensaje a imprimir.

Pero ¿quién es este procesador principal? En mi caso, cualquier clase que cumpla con una **interface** con, por ahora, dos métodos:
ejecutar SQL y ejecutar función.
	
Así que para hacer las pruebas unitarias, simplemente tengo que definir una clase que cumpla con la interface y vaya almacenando las
sentencias lanzadas con sus parámetros y compararlas con un resultado.
	
Por ahora, mis pruebas unitarias consisten en métodos que recogen dos archivos: el script de entrada y los resultados previstos.

Para hacer la prueba, simplemente se carga el script, se ejecuta y se comprueba que los resultados ejecutados sean los mismos que
los resultados previstos.
	
En este momento aún es sencillo, los scripts de pruebas son bastante pequeños pero no sé si evolucionará adecuadamente para
scripts más grandes o si necesito probar realmente scripts más grandes o sólo pequeños trozos de programas.
	
¿Alguna idea sobre esta cuestión en particular?

## Final

Pues gracias por llegar hasta aquí.

Aún estoy trabajando en ello, supongo que estos quince días pasará por todas las pruebas necesarias, corregiré los flecos y comenzaré
el transpilador.
	
Hasta entonces, perdonadme que aún no muestre el código, de alguna forma desconocida he acabado con tres intérpretes:
uno para sentencias SQL normales, otro para ETLs y otro para scripts del nuevo lenguaje (que por ahora he llamado SQLx). Un poco de
limpieza no le va a venir mal antes de salir al público.