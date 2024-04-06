+++
title = "Estándares de código (y llaves)"
date = "2024-04-06"
description = "Opinión sobre los estándares de código y las llaves"
tags = [ "Opinión" ]
+++

Tengo que reconocer que no soy el mayor seguidor de los estándares de código y algunos de ellos me ponen muy nervioso. 

¿Quieres que hablemos de alguno de ellos? Pues hablemos de la obligación de escribir llaves en los bloques de código aunque tengan únicamente una línea. 

Es decir, este código:

```csharp
if (a > 0)
	Console.WriteLine("Es mayor que 0");
else
	Console.WriteLine("Es menor o igual que 0");
```

que verás constantemente en mi GitHub o en este sitio web, en realidad según los últimos estándares de no sé quien debería ser algo así:

```csharp
if (a > 0)
{
	Console.WriteLine("Es mayor que 0");
}
else
{
	Console.WriteLine("Es menor o igual que 0");
}
```

¿Por qué no utilizo llaves? por legibilidad. Si tengo un código con varios `if` y `for` con una única sentencia en su interior y a todos ellos les pongo
llaves de inicio y fin, acabo teniendo un código con más llaves que sentencias ejecutables. 

¿Y por qué eso me pone tan nervioso? Si mi pantalla de Visual Studio ahora mismo muestra 49 líneas de código (sí, lo he mirado) me interesa leer en esas líneas
la mayor cantidad de código posible para comprender la estructura global del código. Si la mayor parte de esas líneas son llaves (en el código anterior, el 50%), 
me obliga a utilizar el scroll arriba y abajo constantemente y acabo perdiendo la noción de dónde estoy.

La cosa, por supuesto, puede ir a peor:

```csharp
public void Metodo()
{
	for (int a = -5; a < 5; a++)
	{
		if (a > 0)
		{
			Console.WriteLine("Es mayor que 0");
		}
		else
		{
			Console.WriteLine("Es menor o igual que 0");
		}
	}
}
```

Pero esas son mis razones. Y aparentemente lo hago mal. 

No importa si en el lenguaje de programación que utilizo (C# en este momento) se está dedicando un tiempo
considerable a eliminar las llaves innecesarias con funciones sin cuerpo (que deben tener otro nombre que no recuerdo), espacios de nombres sin llaves o
`switch` en línea sin `break` que no tiene que ver con las llaves pero se dirigen al mismo problema: reducir el número de líneas de código.

No importa tampoco si el equipo que definió el lenguaje se tomó la molestia de hacer opcionales la llaves en casos de una única sentencia, con lo que ello implica 
a la hora de desarrollar el compilador. 

En aras de una mayor estandarización del código y por mucho que me pese, lo mejor es utilizar llaves.

Y el caso es que cuando pregunto porqué debo escribir llaves innecesarias (y para mí molestas aunque eso es opinable), todos me hablan del error "gotofail" de iOS 
en los métodos de interpretación de los códigos SSL/TLS, algo que ocasionó un problema de seguridad bastante importante y que se describe por ejemplo
en este artículo de [ImperialViolet](https://www.imperialviolet.org/2014/02/22/applebug.html).

El caso es que si miramos el código responsable del error tal como aparece en 
[Apple open source](http://opensource.apple.com/source/Security/Security-55471/libsecurity_ssl/lib/sslKeyExchange.c) nos encontramos con este método:

```c
static OSStatus
SSLVerifySignedServerKeyExchange(SSLContext *ctx, bool isRsa, SSLBuffer signedParams,
                                 uint8_t *signature, UInt16 signatureLen)
{
    OSStatus        err;
    SSLBuffer       hashOut, hashCtx, clientRandom, serverRandom;
    uint8_t         hashes[SSL_SHA1_DIGEST_LEN + SSL_MD5_DIGEST_LEN];
    SSLBuffer       signedHashes;
    uint8_t            *dataToSign;
    size_t            dataToSignLen;
 
    signedHashes.data = 0;
    hashCtx.data = 0;
 
    clientRandom.data = ctx->clientRandom;
    clientRandom.length = SSL_CLIENT_SRVR_RAND_SIZE;
    serverRandom.data = ctx->serverRandom;
    serverRandom.length = SSL_CLIENT_SRVR_RAND_SIZE;
 
 
    if(isRsa) {
        /* skip this if signing with DSA */
        dataToSign = hashes;
        dataToSignLen = SSL_SHA1_DIGEST_LEN + SSL_MD5_DIGEST_LEN;
        hashOut.data = hashes;
        hashOut.length = SSL_MD5_DIGEST_LEN;
 
        if ((err = ReadyHash(&SSLHashMD5, &hashCtx)) != 0)
            goto fail;
        if ((err = SSLHashMD5.update(&hashCtx, &clientRandom)) != 0)
            goto fail;
        if ((err = SSLHashMD5.update(&hashCtx, &serverRandom)) != 0)
            goto fail;
        if ((err = SSLHashMD5.update(&hashCtx, &signedParams)) != 0)
            goto fail;
        if ((err = SSLHashMD5.final(&hashCtx, &hashOut)) != 0)
            goto fail;
    }
    else {
        /* DSA, ECDSA - just use the SHA1 hash */
        dataToSign = &hashes[SSL_MD5_DIGEST_LEN];
        dataToSignLen = SSL_SHA1_DIGEST_LEN;
    }
 
    hashOut.data = hashes + SSL_MD5_DIGEST_LEN;
    hashOut.length = SSL_SHA1_DIGEST_LEN;
    if ((err = SSLFreeBuffer(&hashCtx)) != 0)
        goto fail;
 
    if ((err = ReadyHash(&SSLHashSHA1, &hashCtx)) != 0)
        goto fail;
    if ((err = SSLHashSHA1.update(&hashCtx, &clientRandom)) != 0)
        goto fail;
    if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
        goto fail;
    if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
        goto fail;
        goto fail;
    if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
        goto fail;
 
    err = sslRawVerify(ctx,
                       ctx->peerPubKey,
                       dataToSign,                /* plaintext */
                       dataToSignLen,            /* plaintext length */
                       signature,
                       signatureLen);
    if(err) {
        sslErrorLog("SSLDecodeSignedServerKeyExchange: sslRawVerify "
                    "returned %d\n", (int)err);
        goto fail;
    }
 
fail:
    SSLFreeBuffer(&signedHashes);
    SSLFreeBuffer(&hashCtx);
    return err;
 
}
```

que si no queréis leerlo completo (que lo entiendo), os resalto la sección. Fijáos en concreto en la línea 9:

```c
if ((err = ReadyHash(&SSLHashSHA1, &hashCtx)) != 0)
    goto fail;
if ((err = SSLHashSHA1.update(&hashCtx, &clientRandom)) != 0)
    goto fail;
if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
    goto fail;
if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
    goto fail;
    goto fail;
if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
    goto fail;
```

En ese código, el segundo `goto` se va a ejecutar siempre, independientemente de la condición `if` saltándose la ejecución del resto del código.

No sé porqué, las mayores críticas al código fueron sobre la falta de llaves alrededor de la sentencia `if` y todos parece que coincidieron en la conclusión:

> Pon siempre llaves alrededor de los bloques de código de las sentencias `if` y esto nunca te sucederá.

Siento ser yo quien te lo diga, pero poner llaves alrededor de ese código es bastante posible que no solucione nada:

```c
if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
{
    goto fail;
}
    goto fail;
```

El problema en este caso no son las llaves (ni los `goto` por mucho que me pese): es mala programación provocada por un desarrollador poco atento,
por falta de pruebas y por tener un compilador que no avisa de que en ese método existe código inalcanzable que nunca se va a procesar.

La respuesta real es que esa serie de bloques `if` deberían resolverse de otra forma. Por ejemplo así (sin llaves):

```c
if ((err = ReadyHash(&SSLHashSHA1, &hashCtx)) != 0 ||
	(err = SSLHashSHA1.update(&hashCtx, &clientRandom)) != 0 ||
	(err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0 ||
	(err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0 ||
	(err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
    	goto fail;
```

**Nota:** por cierto, esto es un examen parcial del error, si os fijáis en el código completo, los mismos bloques de código se copian y pegan
en el mismo método en dos partes diferentes.

En conclusión: si quieres normalizar tu código para poner llaves porque te parece más elegante, más legible, te separa mejor los bloques o cualquier razón
que se te ocurra, me parece perfecto, pero no lo hagas por las razones equivocadas: no pienses que tener llaves te proporciona una mayor red de seguridad
ante los errores. No es cierto.