+++
title = "Roslyn 2: funcionamiento de un compilador"
date = "2019-10-11"
description = "Introducción al funcionamiento de los compiladores antes de profundizar en Roslyn"
thumbnail = "/articles/development/roslyn-2/compilador.jpg"
tags = [ "Programación" ]
+++

Como comentábamos en el [artículo anterior](/blog/articles/development/roslyn-1/roslyn-1) lo que nos ofrece 
**Roslyn** son servicios de compilación, es decir una API completa con métodos de acceso a los procesos internos de compilación.

Antes de explicar estos conceptos quizá lo más adecuado sea recordar qué es un compilador y los diferentes 
procesos de una aplicación de este tipo.

Un compilador es una aplicación que recoge una serie de archivos escritos en un lenguaje de programación y 
los traduce a una serie de archivos que un ordenador puede ejecutar. A los archivos de entrada los llamamos 
código fuente. A los archivos de salida los denominamos codigo objeto.

![Esquema de un compilador](/blog/articles/development/roslyn-2/compilador.jpg "Esquema de un compilador")

Aunque cuando hablamos de compiladores éste es el proceso habitual, puede que el código objeto no sea directamente 
un ejecutable si no un código intermedio como los bytecode de Java o el MSIL de.Net o bien código que un ordenador 
no puede ejecutar directamente, como por ejemplo los archivos CSS obtenidos al procesar un archivo de SASS. En estos 
casos solemos hablar de **traductores** en lugar de compiladores.

De hecho, lo habitual es que en lugar de escribir compiladores que escriban directamene código objeto, se escriban traductores 
que produzcan código intermedio como fuente para otro traductor que finalmente genera código objeto. Así conseguimos reaprovechar 
el código, es decir, si tenemos que escribir traductores de los lenguajes A y B al código máquina de los ordenadores Alfa y Beta 
lo más lógico es escribir el traductor de lenguaje A al código intermedio y el traductor de lenguaje B al código intermedio y 
posteriormente los traductores de código intermedio al lenguaje máquina Alfa y Beta. Así tenemos procesos que nos permiten 
traducir de A a Alfa y Beta y de B a Alfa y Beta. Si posteriormente añadimos un traductor de lenguaje C a código intermedio 
no tenemos que reescribir los traductores a Alfa y Beta. Del mismo modo, si añadimos un traductor de código intermedio al 
lenguaje máquina Delta no tenemos que modificar los traductores de A, B y C.

Los **intérpretes** son un caso especial de traductores que no generan código objeto. Lo que hacen es ejecutar las instrucciones 
del código fuente al mismo tiempo que traducen el código.

Por su parte, si nos centramos en el código fuente todos los lenguajes de programación son lenguajes formales, es decir, 
lenguajes que definen una gramática normalizada o bien formada que elimina la redundancia y las inconsistencias de los 
lenguajes no formales como el humano. 

Así sabemos que en C# después de un `if` nos vamos a encontrar con un paréntesis de apertura, una expresión condicional 
y un paréntesis de cierre. Si en un programa nos encontramos con unas comillas detrás de una instrucción *if* estamos 
ante un error sintáctico. Para definir estas reglas se utilizan metodologías como 
[BNF](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_Form). Una vez definida la gramática 
BNF de nuestro lenguaje de programación, podemos comenzar a trabajar en nuestro traductor.

Los traductores suelen dividir su trabajo en fases de forma que la salida de una fase se convierte en la entrada de la 
siguiente. En concreto solemos hablar de cuatro fases:

* **Análisis léxico:** en esta fase se recogen uno por uno los caracteres del código fuente y se obtiene una cadena de **tokens**. 
Un token es el bloque mínimo de información de una estructura BNF. Así tendremos tokens que identifican un comando *if* 
o un paréntesis de apertura o un nombre de variable o una cadena.

* **Análisis sintáctico:** la segunda fase del proceso recoge la cadena de tokens y crea un árbol sintáctico con instrucciones. En 
esta fase se identifican los errores del tipo 'tras un if debe haber un paréntesis de apertura'.

* **Análisis semántico:** la última fase del proceso de análisis se dedica a crear un árbol  con estructuras complejas agrupando 
las instrucciones en clases y métodos. En esta fase se identifican además los errores semánticos o de significado del 
tipo 'no se puede definir un espacio de nombres dentro de una clase'.

* **Generación de código:** en esta fase, a partir del árbol semántico del proceso anterior, se crean los archivos de 
código objeto.

A partir de la fase de análisis sintáctico se crea además una estructura conocida como **tabla de símbolos** 
donde se almacenan las diferentes variables y tipos. Por supuesto, también se lleva un control sobre los errores 
generados en la lectura del código fuente.

En algunos casos además se pasa por una **fase de optimización** que recoge el código generado y lo procesa 
siguiendo diferentes métodos para que se ejecute más rápidamente.

En definitiva, podemos decir que un traductor sigue esta estructura básica:

![Fases de un compilador](/blog/articles/development/roslyn-2/fases-compilacion.jpg "Fases de un compilador")

Aparte de la generación de código final, **Roslyn** nos da acceso al árbol sintáctico y al árbol semántico de 
nuestro código fuente. Para quien se pregunte qué ocurre con la fase de análisis léxico, la cadena de tokens 
resultante no ofrece demasiada información útil aunque **Roslyn** la incluye en ciertos nodos del árbol sintáctico.

Esto será lo que veremos en el próximo artículo.﻿