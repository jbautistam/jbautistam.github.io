+++
title = "Expresiones lógicas"
date = "2024-04-21"
description = "Expresiones lógicas"
thumbnail = "/images/noimage.jpg"
tags = [ "Desarrollo" ]
+++

He escrito sobre la interpretación de [árboles de expresiones](/blog/articles/source-code/calculo-expresiones/calculo-expresiones) en esta web en un par de ocasiones. Suele ser un
proceso no excesivamente difícil pero sí bastante laborioso.

En estos últimos meses he recuperado una forma sencilla de evaluar expresiones lógicas que sirve para casos
muy concretos y siempre me ha parecido muy inteligente. Por supuesto, la idea no es mía y tendrá un
nombre que no recuerdo. Si alguno lo sabe, no dudéis en comentarlo.

El caso de uso para el que lo he estado utilizando es una herramienta de gestión de correos electrónicos. La
idea es que a partir de ciertas palabras o frases del cuerpo del mensaje lo derivemos al correo correspondiente:
un sistema de reglas de correo básico para entendernos.

Por ejemplo, si en el cuerpo de correo aparece la palabra *factura* lo derivamos a la lista de correo de facturación pero
si en el correo aparece la palabra *factura* y además aparece la palabra *reclamación* lo derivamos a la lista de correo
de incidencias.

La idea es por tanto tratar fórmulas de este tipo:

```csharp
("factura" or "reclamación") and ("cliente") and not ("pedido" or "albarán")
```

¿Cuál va a ser el método para evaluar esta regla?

El primer paso va a ser comparar las cadenas que tenemos en la regla con las cadenas del cuerpo del correo y vamos a sustituir
las cadenas que encontremos por un `1` y las que no encontremos, la sustituiremos por un `0`.

Así, por ejemplo si tenemos este cuerpo de correo:

```
El cliente solicita que se le envíe la factura del 2 de abril de 2.020
```

al ejecutar la comparación, el resultado es el siguiente:

```csharp
(1 or 0) and (1) and not (0 or 0)
```

¿Os imagináis cómo podemos seguir?

Lo único que nos queda es sustituir de esta forma:

* Reemplazamos `0 or 0` por `0`
* Reemplazamos `0 or 1` por `1`
* Reemplazamos `1 or 1` por `1`
* Reemplazamos `1 and 0` por `0`
* Reemplazamos `1 and 1` por `1`
* Reemplazamos `not 0` por `1`
* Reemplazamos `(0)` por `0`
* ...

Por supuesto, hay más casos pero supongo que se entiende la idea. Así, tras sucesivos reemplazos tenemos las siguientes cadenas:

```csharp
(1) and 1 and not (0)
1 and 1 and not 0
1 and 1
1
```

Ya no se pueden hacer más sustituciones, el resultado es un `1` que indica que la expresión se ha evaluado a `true`.

Así de fácil.

## Código de evaluación

Pero vamos directamente a ver el código:

```csharp
/// <summary>
///		Motor de ejecución de fórmulas
/// </summary>
public class Interpreter
{
	/// <summary>
	///		Ejecuta una fórmula comprobando una cadena
	/// </summary>
	public bool Execute(string formula, string source, char quote = '"')
	{
		string result = string.Empty;

			// 1. Normaliza la fórmula y el origen: quita los saltos de línea, tabuladores y los espacios dobles
			formula = Normalize(formula);
			source = Normalize(source);
			// 2. Si realmente nos queda algo por analizar
			if (!string.IsNullOrWhiteSpace(formula) && !string.IsNullOrWhiteSpace(source))
			{
				// 2.1. Comprueba si las cadenas de la regla (formula) están el valor buscado (source) y las reemplaza por 0s y 1s
				result = CheckStringsAtSource(formula.ToLower(), source.ToLower(), quote);
				// 2.2. Interpreta las condiciones
				result = Parse(result);
			}
			// 3. Normaliza el resultado
			if (string.IsNullOrWhiteSpace(result))
				result = "0";
			else
				result = result.Trim();
			// 4. Devuelve el valor que indica si se ha ejecutado correctamente
			return result.Equals("1");
	}

	/// <summary>
	///		Normaliza una cadena: quita saltos de línea y espacios dobles
	/// </summary>
	private string Normalize(string value)
	{
		string[] skipChars = ["\r", "\n", "\t", "  "];
		bool replaced;

			// Quita los caracteres extraños: saltos de línea, tabuladores, dobles espacios
			do
			{
				// Indica que por ahora no se ha reemplazado nada
				replaced = false;
				// Reemplaza los caracteres especificados
				foreach (string skipChar in skipChars)
					if (!string.IsNullOrWhiteSpace(value) && value.IndexOf(skipChar) >= 0)
					{
						// Reemplaza el valor
						value = value.Replace(skipChar, " ");
						// Indica que se ha reemplazado el valor
						replaced = true;
					}
			}
			while (!string.IsNullOrWhiteSpace(value) && replaced);
			// Devuelve la cadena normalizada
			return value;
	}

	/// <summary>
	///		Comprueba si las cadenas de la fórmula (lo que haya entre comillas), se encuentran en la cadena de origen.
	///		Si una cadena está en el origen, la cambia por 1, si no, la cambia por 0
	/// </summary>
	private string CheckStringsAtSource(string formula, string source, char quote)
	{
		string result = string.Empty, readedString = string.Empty;
		bool atQuotes = false;
		char previousChar = '#';

			// Recorre los caracteres buscando lo que haya entre comillas
			for (int index = 0; index < formula.Length; index++)
			{
				char actualChar = formula[index];

					// Si estamos en una cadena, guardamos hasta que encontremos el final de la cadena
					if (atQuotes)
					{
						if (actualChar == quote && previousChar != '\\') // ... si estamos entre comillas, la siguiente comilla cierra la cadena a menos que el carácter anterior sea \
						{
							// Comprobamos si la cadena está en el origen
							if (string.IsNullOrWhiteSpace(readedString) || source.IndexOf(readedString) >= 0) // ... si la cadena buscada está en el origen
								result += "1";
							else
								result += "0";
							// Ya no estamos entre comillas
							atQuotes = false;
							readedString = string.Empty;
						}
						else // estamos entre comillas y por tanto guardamos este carácter
							readedString += actualChar;
					}
					else if (actualChar == quote) // ... si encontramos una comilla, es porque se arranca una cadena
						atQuotes = true;
					else // ... estamos fuera de las comillas, guardamos la cadena en el resultado
						result += actualChar;
					// Guarda el carácter anterior
					previousChar = actualChar;
			}
			// Devuelve la cadena resultante (que ya sólo debe estar formadas por 1s y 0s, operadores lógicos y paréntesis)
			return result;
	}

	/// <summary>
	///		Interpreta la fórmula: la fórmula debería ser una cadena del tipo "(1 or 0) and (not 1 or 0)", es decir, 1s y 0s, operadores
	///	de relación y paréntesis
	/// </summary>
	private string Parse(string formula)
	{
		int loops = 0;

			// Normaliza la cadena (no debería haber saltos pero sí que puede que tengamos espacios dobles)
			formula = Normalize(formula);
			// Interpretamos la cadena
			while (!string.IsNullOrWhiteSpace(formula) &&  formula.Length > 1 && loops < 1_000)
			{
				// Quita los paréntesis que hayan podido quedar con espacios
				formula = formula.Replace("( ", "(");
				formula = formula.Replace(" )", ")");
				// Calcula los OR
				formula = formula.Replace("1 or 1", "1");
				formula = formula.Replace("0 or 1", "1");
				formula = formula.Replace("1 or 0", "1");
				formula = formula.Replace("0 or 0", "0");
				// Calcula los AND
				formula = formula.Replace("1 and 1", "1");
				formula = formula.Replace("0 and 1", "0");
				formula = formula.Replace("1 and 0", "0");
				formula = formula.Replace("0 and 0", "0");
				// Calcula los NOT
				formula = formula.Replace("not 1", "0");
				formula = formula.Replace("not 0", "1");
				// Calcula los paréntesis
				formula = formula.Replace("(0)", "0");
				formula = formula.Replace("(1)", "1");
				// Incrementamos el número de bucles (para no complicarnos demasiado, suponemos que más de 1.000 pasos por el bucle es porque hay error)
				loops++;
			}
			// Devuelve la cadena
			return formula;
	}
}
```

El código (por supuesto, mejorable) es bastante más sencillo que la interpretación de los árboles de expresiones habitual. 

Para no complicarlo, me he saltado la comprobación de errores de escritura, simplemente se sale de la interpretación cuando 
se pasa de más de 1.000 iteraciones si no hemmos llegado a una cadena de un carácter de longitud. Por supuesto, se podría
mejorar comprobando si hay en la cadena cualquier cadena que no pertenezca al rango de tokens válidos (and, or, not, 0, 1 o paréntesis) o
bien comprobando simplemente si ninguno de los reemplazos ha conseguido sustituir algo en la cadena (así nos evitamos errores del
tipo `1 and or 1`).