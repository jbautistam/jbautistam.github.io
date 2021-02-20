+++
title = "Métodos criptográficos de clave pública"
date = "2019-10-11"
description = "Explicación de los algoritmos criptográficos de clave pública"
thumbnail = "/articles/seguridad/clave-publica/criptografia-clave-publica.jpg"
tags = [ "Programación", "Seguridad" ]
+++

La criptografía de clave pública, también conocida como asimétrica, se basa en la utilización de
un par de claves.
	
De esta pareja de claves una se utiliza para cifrar y la otra para descifrar. La primera de ellas
se conoce como clave pública y la otra como clave privada.
	
Las dos claves se relacionan utilizando una función matemática que se calcula a partir de la propia
función y de su función inversa. Esta función inversa debe ser difícil o imposible de calcular a partir
de las claves.
	
Matemáticamente esta función se define como:

* x -> A, f(x)
* y -> f(A), x = f-1(y)
	
La función f(x) debe ser fácil de calcular para que el proceso de encriptación sea lo más sencillo posible. Además
f(x) debe ser una función unidireccional, es decir, a partir del resultado no se puede obtener el parámetro de entrada,
en nuestro caso, el texto encriptado.

Existen varios algoritmos para describir estas funciones, aquí veremos tres de ellos

## Algoritmo de Diffie-Hellman

Desarrollado en 1976 por los matemáticos Whitfiled Diffie y Martín Hellman no es realmente un algoritmo de encriptación
dado que se diseñó para poder intercambiar las claves privadas en los cifrados simétricos.

El algoritmo sigue estos pasos:

1. Se selecciona un valor primo 'p' y un generador g-> Z* p. Los valores g y p son valores públicos. El Z* es 
el conjunto de números enteros menores que el valor 'p' que además son primos relativos a éste.
2. El emisor que desea enviar una clave privada, escoge un valor x -> Zp - 1 aleatoriamente, calcula el valor
X = gx mod p y envía este valor X al receptor.
3. El receptor escoge un valor y -> Zp - 1 aleatoriamente y calcula el valor Y = gy mod p. Una vez calculado
se lo envía al emisor.
4. El emisor calcula la clave de esta forma: K = (gy mod p)x mod p.
5. El receptor calcula la clave de esta forma: K = (gx mod p)y mod p.
	
En ambos casos, el valor de K es el valor de la clave.

## Cifrado RSA - Rivest Shamir Adleman

El método de criptografía RSA es un método asimétrico por bloques.

El algoritmo utiliza los siguientes pasos:

1. Se calcula un número 'N' multiplicando dos números primos (muy grandes) 'p' y 'q'. Obteniendo como
resultado N = p · q y Φ(N) = (p-1) · (q-1)
2. Se escoge un número 'e' tal que 1 < e < Φ(N) y el MCD (e, Φ(N)) = 1 ('e' y Φ(N) son primos entre sí).
3. Calculamos el número inverso de 'e0', llamado 'd', de forma que e · d = 1 (mod Φ(N)).
	
Hasta ahora hemos obtenido ambas claves, 'e' y 'd', donde la clave pública es (e, N) y la clave privada es (d, N).

Ahora para cifrar obtenemos C de forma que:

* C = Me (mod N) con MCD (M, N) = 1 y M < N
	
y para descifrar:


* M = Cd (mod N)
	
El ratio de seguridad de este tipo de cifrado es bastante alto, dado que no hay una forma fácil y rápida
de obtener los factores primos de un número grande utilizando los ordenadores actuales.

## Cifrado Elgamal

El cifrado Elgamal se basa en el algoritmo explicado previamente para el intercambio de claves **Diffie - Hellman**.

Lo documentó inicialmente Taher Elgamal en 1984 y actualmente se utiliza en varias versiones de PGP como
GNU Privacy Guard (GPG).
	
Para realizar la codificación, seguimos los siguientes pasos:

1. El receptor obtiene un valor numérico primo grande llamado 'p' y una semilla g -> Z*p y aleatoriamente
un valor 'xr' de modo que 0 < xr < p-1. En este caso 'xr' se considera la clave privada.
2. La clave pública se obtiene como: yr = gxr mod p.
3. El emisor obtiene un valor 'K' aleatoriamente de modo que MCD(K, p-1) = 1. En este caso 'K' es un primo
relativo a p-1.
4. El emisor calcula r = gK (mod p) y s = M · yKr (mod p).
	
Al cifrar, el emisor manda el mensaje como:

* C = (r, s), donde 'r' es la clave pública del emisor y 's' es el mensaje cifrado.
	
Para descifrar el receptor calcula:

* M = s / rxr.
	
La fortaleza de este método criptográfico se basa en la dificultad del cálculo de logaritmos
discretos necesarios para descifrar el mensaje.