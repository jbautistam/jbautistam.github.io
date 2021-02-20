+++
title = "Contadores rendimiento"
date = "2019-10-11"
description = "Creación de contadores de rendimiento con .NET"
thumbnail = "/articles/development/contadores-rendimiento/contadores-rendimiento.jpg"
tags = [ "Programación", "Sistema" ]
+++

Los contadores de rendimiento proporcionan información numérica sobre el estado de una operación del sistema operativo o
una aplicación. Estos contadores se pueden grabar y monitorizar utilizando herramientas estándar del sistema 
(concretamente la aplicación de rendimiento de Windows que puede ejecutarse con la orden: perfmon.msc /s).

Aparte de visualizar los contadores del sistema, desde.NET podemos crear **contadores de rendimiento** propios 
para nuestras aplicaciones (siempre que tengamos los permisos de administración necesarios) así como consultar los datos de
rendimiento o eliminarlos.

El resto del artículo se centra en las diferentes acciones relacionadas con los **contadores de rendimiento** a las que podemos
acceder desde.NET:

## Comprobar si existe un contador de rendimiento

Cada **contador de rendimiento** pertenece a una categoría. Tanto la categoría como sus contadores se crean al mismo tiempo,
no es posible añadir contadores a una categoría existente. 

Antes de crear una categoría se debería comprobar si existe, si es así, no se puede crear otra con el mismo nombre (el resultado
será una excepción).

Para comprobar si existe una categoría se puede llamar el método estático **Exists** de la clase **PerformanceCounterCategory**.
El valor de retorno nos indica si existe o no la categoría:

```csharp
bool blnExists = PerformanceCounterCategory.Exists("Categoria");
```

## Creación de un contador de rendimiento entero

Uno de los tipos de contadores de rendimiento más habituales es del tipo de datos **NumberOfItems32**. Este tipo de contador
permite almacenar un valor entero de 32bits cada vez que se necesite. Si lo que deseamos es crear un único contador de este
tipo en una categoría lo único que necesitamos es utilizar el método **Create** de la clase **PerformanceCounterCategory**.

La primera versión de este método acepta cinco argumentos: los dos primeros son cadenas para dar nombre y un texto descriptivo
a la categoría, el tercero indica si se va a utilizar la categoría con un único contador o varios, los últimos dos
argumentos se utilizan para añadir un nombre y un texto descriptivo para el contador.

En el siguiente código utilizamos el método **Create** para añadir un contador de rendimiento en la categoría "Facturas"
con el contador "Ventas facturadas". Después seleccionamos el contador de rendimiento donde introducimos un valor:

```csharp
PerformanceCounterCategory.Create("Facturas",
						"Contadores de rendimiento para facturación",
						PerformanceCounterCategoryType.SingleInstance,
						"Ventas facturadas",
						"Valor de ventas facturadas");

PerformanceCounter objCounter = new PerformanceCounter("Facturas", "Ventas facturadas", false);
objCounter.RawValue = 50;
```

Normalmente los **contadores de rendimiento** deberían crearse durante la instalación y asignar los valores cuando se ejecuta
el programa. Esto nos asegura que el contador de rendimiento ya se ha dado de alta en el momento que se va a añadir un valor.

## Creación de una categoría de contadores múltiples

Aparte de las categorías con contadores únicos también se pueden crear categorías para contadores múltiples. En estas 
categorías se crean conjuntos de contadores sobre el grupo cada uno de ellos con un nombre de instancia. El valor
de cada uno de ellos se puede monitorizar a través de la aplicación de rendimiento.

El siguiente código crea una categoría (con el mismo nombre anterior) y le añade un contador, como el contador se
define con varias instancias podemos añadir más de una sección por cada contador si utilizamos nombres diferentes.
En este caso, se añaden dos instancias al contador "Ventas facturadas" una para la "Zona centro" y otra para
"Zona norte":

```csharp
PerformanceCounterCategory.Create("Facturas",
						"Contadores de rendimiento para facturación",
						PerformanceCounterCategoryType.MultiInstance,
						"Ventas facturadas",
						"Valor de ventas facturadas");

PerformanceCounter objCounterCenter = new PerformanceCounter("Facturas", "Ventas facturadas", "Zona centro", false);
objCounterCenter.RawValue = 50;
PerformanceCounter objCounterNorth = new PerformanceCounter("Facturas", "Ventas facturadas", "Zona norte", false);
objCounterNorth.RawValue = 80;
```

## Creación de otros tipos de contadores de rendimiento

Si se desea crea más de un contador de rendimiento en un crupo o necesitamos otros tipos de contadores aparte de
**NumberOfItems32**, se puede utilizar una versión sobrecargada del método **Create**. Esta versión requiere
crear una instancia de **CounterCreationDataCollection**, que mantiene los detalles de los contadores.

Para comenzar se debe crear una colección de contadores vacía utilizando el constructor predeterminado:

```csharp
CounterCreationDataCollection objColCounters = new CounterCreationDataCollection();
```

Una vez exista la colección, se le pueden añadir contadores. Cada uno de ellos tiene un nombre y un texto descriptivo igual que
en los métodos anteriores aunque se puede indicar el tipo de datos que se va a utilizar en el monitor. 

En el siguiente ejemplo añade dos contadores a la colección, el primero muestra el número de pedidos mientras que el 
segundo almacena el número de pedidos por segundo. El primero de ellos utiliza el tipo de contador 
**RateOfCountsPerSecond32** para controlar el número de pedidos por segundo.

```csharp
objColCounters.Add(new CounterCreationData("Pedidos", "Número de pedidos",
								PerformanceCounterType.NumberOfItems32));
objColCounters.Add(new CounterCreationData("Pedidos por segundo", "Número de pedidos por segundo",
								PerformanceCounterType.RateOfCountsPerSecond32));
```

Una vez se ha creado y rellenado la colección, utilizamos la versión sobrecargada del método **Create** cambiando el
cuarto parámetro por la colección de contadores creada anteriormente:

```csharp
PerformanceCounterCategory.Create("Rendimiento pedidos",
					    "Contadores de rendimiento para los pedidos",
					    PerformanceCounterCategoryType.SingleInstance,
					    objColCounters);
```

Para asignar valores a los contadores creados se puede utilizar un método similar al anterior:

```csharp
PerformanceCounter objOrders = new PerformanceCounter("Rendimiento pedidos", "Pedidos", false);
objOrders.RawValue = 253912;

PerformanceCounter objOrdersSecond = new PerformanceCounter("Rendimiento pedidos", "Pedidos por segundo", false);
objOrdersSecond.RawValue = 10;
```

## Borrado de categorías de contadores de rendimiento

El método **Delete** de la clase **PerformanceCounterCategory** permite eliminar una categoría simplemente indicando el nombre
de la categoría a eliminar:

```csharp
PerformanceCounterCategory.Delete("Facturas");
```