---
title: Bau Image Prompt Generator
date: 2024-02-03T15:23:49+01:00
---

**Bau Image Prompt Generator** es una aplicación WPF para generación de imágenes con inteligencia artifical utilizando diferentes API.

Actualmente, utiliza la API de [Stable Horde](https://stablehorde.net/). **Stable Horde** utiliza un cluster distribuido para la generación
de imágenes, no es tan rápido como otras APIs pero proporciona una gran cantidad de modelos de forma gratuita siempre que no te importe
esperar.

Al abrir la aplicación se nos muestra un archivo vacío:

![Bau Image Prompt Generator](/docs/prompt-image/images/bau-prompt-images-generator-1.png)

En la parte superior está la barra de herramientas donde podemos crear un archivo, abrir un archivo existente, grabar las modificaciones o
añadir / quitar versiones al archivo.

Al crear una versión de archivo, se copian todos los datos para permitirnos hacer modificaciones o probar con otros modelos.

En la sección de *Prompt* vemos las opciones para la creación de imágenes.

* **Generator:** es el generador que vamos a utilizar. La aplicación está pensada para utilizar diferentes generadores aunque por ahora sólo
existen dos: *Horde* para generación de imágenes utilizando **Stable Horde** y *Fake* que realmente es un generador que utilizo para pruebas
y que simplemente descarga un par de imágenes (es decir, no es un generador real).
* **Models:** es la lista de modelos que podemos utilizar en el generador seleccionado para crear nuestras imágenes.
* **Positive:** es el texto positivo que se va a utilizar para crear la imagen, es decir el texto que define nuestra imagen.
* **Negative:** es el texto negativo, es decir, lo que no queremos que aparezca en la imagen. Normalmente se utiliza para ajustar y evitar
errores en la imagen creada.

En la sección de *Parameters* se encuentran los parámetros de generación:

* **Widht y Height:** ancho y alto de las imágenes generadas.
* **Images to generate:** es el número de imágenes que deseamos crear al mismo tiempo.
* **Steps:** es el número de pasos que vamos a utilizar para generar la imagen.
* **Cfg scale:** es cómo queremos que la IA interprete nuestro prompt, cuánto más alto sea este valor, más se ajustará la imagen al prompt.
* **Denoising strenght:**
* **Sampler:** sampleador utilizado.
* **Postprocessing:** herramientas de postproceso a utilizar.
* **Karras:** 

![Bau Image Prompt Generator](/docs/prompt-image/images/bau-prompt-images-generator-2.png)

![Bau Image Prompt Generator](/content/docs/prompt-image/images/bau-prompt-images-generator-3.png)

