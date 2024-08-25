+++
title = "Librería 2FA en C#"
summary = "Librería para generación de claves HOTP y TOTP para aplicaciones de autorización"
date = "2024-08-23"
tags = [ "Aplicaciones", "Seguridad" ]
+++

Las aplicaciones de **autenticación 2FA** ofrecen una capa adicional de seguridad para proteger cuentas y recursos digitales.

Existen muchas aplicaciones de este tipo como [Microsoft Authenticator](https://www.microsoft.com/es-es/security/mobile-authenticator-app) o
[Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2).

Particularmente yo utilizaba la versión de escritorio de [Authy](https://authy.com/) pero desde hace unos meses ya no desarrolla versiones de ordenador, 
sólo proporciona aplicaciones móviles. No pasa nada, [KeePass](https://keepass.info/) o [KeePassXC](https://keepassxc.org/) o
prácticamente cualquier gestor de contraseñas que se precie tienen esta funcionalidad, pero antes de utilizarlas, me surgió la curiosidad sobre
cómo funcionan este tipo de aplicaciones.

Prácticamente todos los sistemas 2FA utilizan los algoritmos HOTP ([RFC 4226](http://tools.ietf.org/html/rfc4226)) y su evolución, 
TOTP ([RFC 6238](http://tools.ietf.org/html/rfc6238)) para la generación de códigos de un sólo uso.

Un algoritmo de contraseña de uso único (One Time Password u OTP) es un método de autenticación que genera una contraseña única y 
temporal para una sesión. Este código se utiliza una vez y caduca después de un período de tiempo. 

Las características de estas claves son:

* **Temporalidad:** tienen un tiempo limitado de validez, se pueden utilizar dentro de un breve período de tiempo después de su generación.
* **Exclusividad:** cada clave es única y se genera mediante algoritmos avanzados, garantizando que no se repita y que 
solo su dueño pueda acceder a ella.
* **Generación segura:** creadas mediante procesos criptográficos que imposibilitan la adivinación por parte de los atacantes.
* **Fácil implementación:** pueden enviarse a través de varios medios, como mensajes de texto, correos electrónicos o 
aplicaciones dedicadas, facilitando su integración en diferentes sistemas.

Tanto HOTP (HMAC-Based One-Time Password) como TOTP (Time-Based One-Time Password) funcionan compartiendo una clave secreta que define
el servidor de la cuenta y un valor numérico inicial o semilla. 

En HOTP este valor numérico es un contador que está sincronizado entre el servidor y el cliente y se incrementa cada vez que se genera 
un código. En TOTP este valor numérico es el valor numérico de la fecha de generación medido como el número de ticks desde la medianoche
del día 1 de enero de 1970 (es decir, el tiempo Unix), así evitamos tener un contador compartido además del secreto
siempre que ambos sistemas estén sincronizados.

El algoritmo para ambos casos se inicia generando una clave compartida entre el servidor y la aplicación de autenticación. Ambos
sistemas comparten una clave secreta. Esta clave son los códigos QR que veréis en ocasiones al configurar una cuenta con 2FA en un servidor y
que no es más que una cadena de texto normalmente codificada en [Base32](https://es.wikipedia.org/wiki/Base32).

Al solicitar un código de 2FA, la aplicación de autenticación recoge esa clave compartida y el valor numérico (contador o fecha de sistema) y
lo procesa con una función hash HMAC (Hash-based Message Authentication Code). Este función HMAC puede ser Sha1, Sha12 o Sha256 (el habitual
es Sha1).

Una vez generado el código hash, se reduce o trunca para obtener el código OTP de 6 u 8 dígitos que nos muestra la aplicación de autenticación.

Para comprender mejor cómo funcionaban estos códigos, escribi una librería para implementación de códigos HOTP y TOTP: 
[OneTimePassword](https://github.com/jbautistam/OneTimePassword) basándome en en el código de [Otp.Net](https://github.com/kspearrin/Otp.NET) 
de *Kyle Spearrin* que os recomiendo.

### Generación de una clave HOTP

Para generar una clave HOTP utilizaremos la clase `HotpGenerator`. En el constructor debemos indicar:

* `Key`: clave devuelta por el servidor de claves.
* `Encoding`: modo de codificación de la clave devuelta por el servidor de claves (texto plano, Base64 o Base32).
* `Algorithm`: algoritmo de hashing utilizado para obtener los códigos resultantes (Sha1, Sah128, Sha256). El valor habitual es SHA1.
* `Digits`: número de caracteres generados por el código (de 6 a 8, lo habitual es 6).

Una vez inicializada la clase, llamando al método `Compute` con el contador adecuado y obtenemos al código de validación:

```csharp
using Bau.Libraries.OneTimePassword;

HotpGenerator hotp = new("KEY", Secret.Encoding.Plain, BaseTokenGenerator.HashAlgorithm.Sha1, 6);

string code = hotp.Compute(19238);
```

### Generación de una clave TOTP

Para generar una clave TOTP utilizaremos la clase `TotpGenerator`. En el constructor debemos indicar:

* `Key`: clave devuelta por el servidor de claves.
* `Encoding`: modo de codificación de la clave devuelta por el servidor de claves (texto plano, Base64 o Base32).
* `Algorithm`: algoritmo de hashing utilizado para obtener los códigos resultantes (Sha1, Sah128, Sha256). El valor habitual es SHA1.
* `Digits`: número de caracteres generados por el código (de 6 a 8, lo habitual es 6).

Una vez inicializada la clase, llamando al método `Compute` obtenemos el código de validación:

```csharp
using Bau.Libraries.OneTimePassword;

TotpGenerator totp = new("KEY", Secret.Encoding.Plain, BaseTokenGenerator.HashAlgorithm.Sha1, 6);

string code = totp.Compute();
```

En este caso, si no le pasamos ninguna fecha, se utiliza la fecha del sistema pero le podemos indicar una fecha en
concreto:

```csharp
string code = totp.Compute(new DateTime(2024, 8, 2, 17, 30, 5));
```

o bien utilizar `long` especificando la fecha Unix (número de ticks desde el 1-1-1970):

```csharp
string code = totp.Compute(1_991_289);
```

Inicialmente, el tiempo de validez de la clave, es de 30 segundos pero lo podemos modificar en cualquier momento:

```csharp
using Bau.Libraries.OneTimePassword;

TotpGenerator totp = new("KEY", Secret.Encoding.Plain, BaseTokenGenerator.HashAlgorithm.Sha1, 6);

totp.TimeManager.IntervalSeconds = 60;

string code = totp.Compute();
```

### Tiempo de validez de la clave

Los códigos generados por `TotpGenerator` son válidos durante treinta segundos o el intervalo especificado, pero este
tiempo se mide no desde la generación del código si no desde el inicio del intervalo de la fecha especificada. 
Es decir, si generamos el código a las 12:05, el inicio de la ventana de generación será 12:00 y nos quedarán veinticinco segundos de validez.

Para comprobar el tiempo que nos queda de validez del código (por ejemplo para mostrarlo en una aplicación) podemos utilizar
el método `GetRemainingSeconds` de la clase `TopTimeManager` donde se agrupan los métodos relacionados con la fecha:

```c#
using Bau.Libraries.OneTimePassword;

TotpGenerator totp = new("KEY", Secret.Encoding.Plain, BaseTokenGenerator.HashAlgorithm.Sha1, 6);

totp.TimeManager.IntervalSeconds = 60;

string code = totp.Compute(DateTime.UtcNow);
int remainingSeconds = totp.TimeManager.GetRemainingSeconds(DateTime.UtcNow);
```

## Aplicación de ejemplo

Para terminar, escribí también una aplicación WPF de ejemplo ([BauOtp](https://github.com/jbautistam/BauOTP)) 
de la librería [OneTimePassword](https://github.com/jbautistam/OneTimePassword).

Funciona como la mayoría de aplicaciones de autenticación 2FA. Su pantalla principal permite administrar diferentes cuentas:

![Ventana principal](/blog/articles/development/onetime-password-2fa-library/images/main-window.png)

Al abrir una de esas cuentas, podemos configurarla y ver los códigos generados tanto con TOTP utilizando la fecha de sistema
como con HOTP utilizando un contador:

![Datos de una cuenta 2FA](/blog/articles/development/onetime-password-2fa-library/images/2fa-account.png)

Esta aplicación no es más que un ejemplo de uso, para utilizarla en producción faltaría implementar una
clave maestra o la encriptación de archivos de claves pero eso os lo dejo a vosotros.