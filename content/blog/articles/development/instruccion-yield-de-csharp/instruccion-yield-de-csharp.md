+++
title = "Instrucción yield de C#"
date = "2019-10-11"
description = "Introducción a la instrucción yield de C#"
thumbnail = "/images/noimage.jpg"
tags = [ "Programación" ]
+++

La palabra clave [yield](http://msdn.microsoft.com/en-us/library/9k7k7cf0.aspx) de C# 
quizá sea una de las instrucciones más útiles del lenguage y al mismo tiempo una de las más desconocidas.

Esta instrucción indica al compilador que el método en el que aparece se va a utilizar dentro de un iterador para que este genere una
clase que implemente el comportamiento del bucle definido en el método.

La instrucción `yield` se utiliza junto a la sentencia `return` para devolver un valor al bloque que realiza la iteración (por
ejemplo un bucle `for each`) o junto a la sentencia `break` para salir de un bucle dentro de un iterador.

Esta instrucción puede utilizarse, entre otras cosas, para evitar crear listas intermedias. Por ejemplo, si tenemos una lista de nombres y deseamos obtener todos
aquellos que comiencen por una letra lo habitual es utilizar este código:

```csharp
private void Print()
{ 
	List<string> objColNames = new List<string> { "Juan", "Antonio", "Pepe", "Ana" };
	List<string> objColFound = FindStarts("A");
	
	foreach (string strName in objColFound)
		Console.WriteLine(strName);
}

private IList<string> FindStarts(IEnumerable<string> objColNames, string strStart)
{ 
	List<string> objColFound = new List<string>();
	
	foreach(string strName in objColNames)
		if(strName.StartsWith("Bob"))
			objColFound.Add(strName);  
	
	return objColFound;
}
```	

La primera función (`PrintStart`) simplemente define una lista de nombres y llama a la segunda para encontrar aquellos que 
comienzan por la letra A. La segunda función (`FindStarts`) busca aquellos nombres que comienzan por la letra pasada como parámetro
y devuelve una lista nueva creada con estos elementos.

Sin embargo, una forma más sencilla de hacer esto es utilizar la palabra clave `yield` como en el siguiente ejemplo:

```csharp
private void Print()
{ 
	List<string> objColNames = new List<string> { "Juan", "Antonio", "Pepe", "Ana" };
	
	foreach (string strName in FindStarts(objColNames, "A")
		Console.WriteLine(strName);
}

private IEnumerable<string> FindStarts(IEnumerable<string> objColNames, string strStart)
{ 
	foreach(string strName in objColNames)
		if(strName.StartsWith("Bob"))
			yield return strName;  
}
```	

Como podemos ver, ya no es necesario crear ninguna lista intermedia si no que es el propio compilador quien genera los elementos de esta lista
cada vez que se pasa por el bucle `foreach`.

**Nota:** como nos podemos imaginar por el ejemplo anterior, `yield` es una de las instrucciones más utilizadas 
internamente en **LinQ**.
	
Ni siquiera es necesario utilizar una lista, también podemos usar simplemente un bucle simple para obtener los resultados de una función. Por
ejemplo, si deseamos ver los números pares entre dos números podemos utilizar una función de este tipo:

```csharp
public static IEnumerable GetEvenNumbers(int intStart, int intEnd)
{	
	for (int intIndex = intStart; intIndex <= intEnd; intIndex++)
		if ((intIndex % 2) == 0)
			yield return intIndex;
}
```

Por supuesto, nuestro bucle puede ser todo lo complejo que deseemos. Se me ocurre por ejemplo una función que itere sobre las palabras de una 
cadena:

```csharp
public static IEnumerable GetWords(string strSentence)
{	
	string strWord = "";
	
	foreach (char chrChar in strSentence)
		if (chrChar == ' ')
		{ 
			yield return strWord;
			strWord = "";
		}
		else
			strWord += chrChar;
}
```

Podemos encontrar ejemplos mucho más complejos por la red, por ejemplo, uno bastante ilustrativo explica cómo tratar 
[pipes y filtros](http://ayende.com/blog/3082/pipes-and-filters-the-ienumerable-appraoch) utilizando la 
sentencia `yield` con muy pocas líneas de código.