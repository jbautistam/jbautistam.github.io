+++
title = "Desarrollo de plugins con .Net"
date = "2019-10-11"
description = "Introducción al desarrollo de plugins en aplicaciones .NET"
thumbnail = "/articles/source-code/plugins/plugins.jpg"
tags = [ "Programación" ]
+++

El concepto de **plugin** o **extensión** es bien conocido para todo desarrollador: un componente dentro de una
aplicación que se puede cargar y utilizar en tiempo de ejecución sin vincularlo directamente en tiempo de diseño.
	
Los plugins nos permiten añadir funcionalidades a nuestra aplicación para las que no estaba preparada en un principio
e incluso cargarlos o descargarlos sin necesidad de configuración.

En.Net, existen varias formas de añadir plugins a un ejecutable. En teoría podemos gestionarlos directamente
utilizando `Reflection` y carga dinámica de ensamblados con el método `LoadAssembly` aunque es mucho más
sencillo si utilizamos las librerías de [MEF](https://msdn.microsoft.com/es-es/library/dd460648(v=vs.110).aspx).

## Introducción a MEF

**MEF** o **Managed Extensibility Frameworks** (no confundir con EF - Entity Frameworks) es una librería
para la creación de aplicaciones que usen extensiones o plugins. Esta librería permite
descubrir o utilizar extensiones evitando la inclusión de dependencias, es decir, sin
añadir referencias a otras librerías dentro de nuestra aplicación principal.

Se puede usar **MEF** prácticamente con cualquier tecnología de.Net, es decir, tanto WPF como Windows Forms, consolas de 
Windows, servicios o ASP.Net, de hecho, se podría usar como sistema de inyección de dependencias aunque le faltan
algunas de las características a las que estamos acostumbrados y que tendríamos que implementar a mano.   

**Nota:** A los programadores que utilicen Visual Studio quizá les suene MEF por una de las ventanas
que aparecen al abrir la aplicación que dice algo así como 'Creando el árbol de componentes MEF'. Cierto, es la misma
tecnología que usa Visual Studio para sus extensiones.

Para utilizar **MEF** simplemente debemos añadir una referencia al ensamblado `System.ComponentModel.Composition`
tanto en la aplicación que vamos a utilizar como host como en las que van a actuar como plugin o extensión y añadir
atributos al código para indicar las clases que se exportan e importan.

**Nota:** para liar un poco las cosas, Microsoft decidió portar **MEF** en un ensamblado similar llamado
`Microsoft.Composition` dedicado a las aplicaciones de Windows Store con añadidos para MVVM. Este ensamblado no es compatible
con el anterior, de hecho el código a utilizar es diferente en ambos casos.
	
Veremos cómo utilizar estas librerías para crear nuestro sistema de plugins en este artículo.

## Definiendo los interfaces de los plugins

**MEF** nos ayuda a descubrir y añadir plugins a nuestras aplicaciones pero no impone ninguna restricción a los interfaces
de comunicación entre las diferentes librerías, de hecho, lo más complicado en este caso es definir los métodos que vamos a utilizar
y crear un sistema robusto que nos permita añadir 'cualquier' tipo de librería a nuestra aplicación principal.
	
Para este artículo, he definido una serie de interfaces básicos tanto para las clases que vamos a utilizar como plugin como
para las clases que van a trabajar como host, es decir, las clases que se van a utilizar desde nuestra aplicación principal.
	
Los interfaces son intencionadamente simples, por supuesto habría que ampliarlos para una aplicación real pero pretendo que
el ejemplo sea lo más sencillo posible para no confundir la base de **MEF** con la implantación en nuestra aplicación.
	
En primer lugar, vamos a ver el interface de la clase Host, es decir, la clase de nuestra aplicación principal con la que se
van a comunicar los plugins:
	
```csharp
namespace Bau.Libraries.LibPlugins.Core.Host
{
	// Interface para la aplicación Host
	public interface IHostController
	{ 
		//  Muestra un mensaje
		void ShowMessage(string strMessage);
	
		// Nombre de la aplicación
		string ApplicationName { get; }
	}
}
```

Un interface simple: una propiedad con el nombre de la aplicación y un método que muestra un mensaje.

El interface de las clases que funcionan como plugins es igual de simple:

```csharp
namespace Bau.Libraries.LibPlugins.Core.Plugins
{
	// Interface para los plugins
	public interface IPlugin
	{
		// Inicializa las librerías del plugin
		void InitLibraries(Host.IHostController objHostController);

		// Devuelve un mensaje
		string GetHelloMessage();

		// Nombre del plugin
		string Name { get; }
	}
}
```

Sólo hay un par de métodos: uno que informa al plugin de cuál va a ser su controlador principal (`InitLibraries`) y otro
utilizado para obtener un mensaje de saludo (`GetHelloMessage`). Por último, le añadimos una propiedad con el nombre del 
plugin (`Name`).
	
En **MEF** tenemos la oportunidad de asociar atributos con metadatos a nuestras clases exportadas, el interface que he
utilizado en este caso para los metadatos es el siguiente:
	
```csharp
namespace Bau.Libraries.LibPlugins.Core.Plugins
{
	// Interface para los datos de un plugin
	public interface IPluginMetaData
	{
		// Nombre del plugin
		string Name { get; }

		// Descripción del plugin
		string Description { get; }
	}
}
``

con dos propiedades para el nombre y la descripción del plugin.

Por facilitar las cosas, también creamos una clase abstracta para los plugins. Esto no es completamente necesario, pero cuando
ampliemos los interfaces, es posible que nos sea de utilidad para implementar métodos comunes para todos los plugins:

```csharp
// Clase base para los plugins
public abstract class AbstractBasePlugin: IPlugin
{
	public AbstractBasePlugin(string strName)
	{ 
		Name = strName;
	}

	// Inicializa las librerías del plugin
	public abstract void InitLibraries(Host.IHostController objHostController);

	// Obtiene el mensaje de saludo
	public abstract string GetHelloMessage();

	// Nombre del plugin
	public string Name { get; }

	// Controlador de la aplicación principal
	public Host.IHostController HostController { get; protected set; }
}
```

Como podemos observar, aún ni siquiera hemos comenzado a utilizar MEF, simplemente hemos creado interfaces y clases abstracta que utilizaremos
como base en nuestras aplicaciones de plugin.
	
**Nota:** estos interfaces se han definido en una librería llamada `LibPlugins.Core` en la solución. Debemos
añadir referencias a esta librería tanto en nuestras librerías de plugin como en las librerías o aplicaciones host para que 
compartan sus métodos.

## Implementando un plugin

Para la implementación de una aplicación como plugin, nos creamos otro proyecto (en este caso lo he llamado `PluginSampleHello`) 
y una clase que implemente el interface `IPlugin`:
	
```csharp
using System;
using System.ComponentModel.Composition;

using Bau.Libraries.LibPlugins.Core.Host;
using Bau.Libraries.LibPlugins.Core.Plugins;

namespace PluginSampleHello
{
	//	Controlador del plugin "SampleHello"
	[Export(typeof(IPlugin))]
	[ExportMetadata("Name", "SampleHello")]
	[ExportMetadata("Description", "Prueba de plugin")]
	public class PluginSampleHelloController: AbstractBasePlugin
	{	
		public PluginSampleHelloController(): base("SampleHello") {}

		// Inicializa las librerías
		public override void InitLibraries(IHostController objHostController)
		{ 
			System.Timers.Timer tmrMessage = new System.Timers.Timer(5000);

			// Inicializa el controlador
			HostController = objHostController;
			// Inicializa un temporizador para enviar un mensaje al controlador
			tmrMessage.Start();
			tmrMessage.Elapsed += (objSender, objEventArgs) =>
				{ HostController.ShowMessage("Mensaje enviado desde PluginSampleHelloController");
				};
			// Muestra el mensaje en la consola
			Console.WriteLine("Inicializando las librerías de SampleHello");
		}

		// Obtiene el mensaje de saludo
		public override string GetHelloMessage()
		{ 
			return $"Hello desde PluginSampleHelloController. Mi host es {HostController.ApplicationName}";
		}
	}
}
```

Siguiendo con la filosofía del artículo, no he hecho más que implementar la interface a través de la clase abstracta, es decir,
hemos definido los métodos `InitLibraries` y `GetHelloMessage`. 
	
Además, al método `InitLibraries` le hemos añadido un temporizador para que cada cierto tiempo mande un mensaje a la aplicación Host 
para así demostrar que podemos llamar tanto desde el plugin al host como desde el host al plugin. 
	
De cualquier forma, ya hemos comenzado a utilizar **MEF** en esta clase añadiendo el atributo `Export`
a la cabecera de la clase indicando que exportamos o implementamos el interface `IPlugin` que definimos en la
sección anterior. 
	
Los otros dos atributos `ExportMetadata` nos sirven para añadir metadatos al plugin que podemos utilizar desde la aplicación
host. Si nos fijamos, son las mismas propiedades que definimos en `IPluginMetaData` y que no hemos implementado en ninguna
clase real.
	
Hasta aquí la configuración del plugin: no hace falta nada más para tener una extensión con **MEF**, simplemente nos
queda pasar a la aplicación que funcionará como host.

## Implementando una aplicación host

La aplicación host es aquella que va a importar y utilizar los plugins. En algunos casos también se encargará de cargarlos y descargarlos o
de las comunicaciones entre ellos.
	
Para implementarlo nos creamos otro proyecto: una aplicación WPF llamada `TestPlugins`. Podríamos haber elegido
cualquier otro tipo de aplicación pero considero que una aplicación de escritorio es más adecuada para este tipo de pruebas.
	
La clase que importa y controla los plugins es la siguiente:

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.Composition;
using System.ComponentModel.Composition.Hosting;

using Bau.Libraries.LibPlugins.Core.Host;
using Bau.Libraries.LibPlugins.Core.Plugins;

namespace TestPlugins.Plugins
{
  // Manager de plugins
  [Export(typeof(IPluginManager))]
  public class PluginsManager: IPluginManager
  {
    #pragma warning disable 0649
    //Plugins
    [ImportMany]
    private IEnumerable<Lazy<IPlugin, IPluginMetaData>> objColPlugins;
    #pragma warning restore 0649

    // Inicializa los datos
    public void Initialize(Type objTypeMainAssembly, string strPathsPlugins)
    { 
    	CompositionContainer objContainer;
		AggregateCatalog objCatalog = new AggregateCatalog();
		string [] arrStrPaths = strPathsPlugins.Split(';');
		
		// Añade todas las secciones localizadas en el ensamblado principal
		objCatalog.Catalogs.Add(new AssemblyCatalog(objTypeMainAssembly.Assembly));
		// Añade los directorios de plugins
		if (arrStrPaths.Length > 0)
			foreach (string strPath in arrStrPaths)
				if (System.IO.Directory.Exists(strPath))
					objCatalog.Catalogs.Add(new DirectoryCatalog(strPath));
		// Crea el CompositionContainer con las secciones del catálogo
		objContainer = new CompositionContainer(objCatalog);
		// Rellena los datos importados de este objeto
		try
		{  
			objContainer.ComposeParts(this);
		}
		catch (CompositionException objException)
		{ 
			Console.WriteLine(objException.ToString());
		}
    }

  // Inicializa los plugins
  public void InitPlugins(IHostController objHostController, out string strError)
  { 
  	// Inicializa los argumentos de salida
    strError = "";
    // Indica a los plugins que inicialicen sus librerías
    if (Plugins != null)
      foreach (Lazy<IPlugin, IPluginMetaData> objPlugin in Plugins)
        try
          { 
          	objPlugin.Value.InitLibraries(objHostController);
          }
        catch (Exception objException)
          { 
          	strError += $"Error en el método InitLibraries del plugin {objPlugin.Metadata.Name}.\nExcepción: {objException.Message}" + Environment.NewLine;
          }
    }

  // Plugins
  public IEnumerable<Lazy<IPlugin, IPluginMetaData>> Plugins 
  { 
  	get { return objColPlugins; }
  }

  // Lista de plugins
  public List<IPlugin> PluginsController
  { get 
      { 
      	List<IPlugin> objColPlugins = new List<IPlugin>();

        // Crea la lista de plugins
        if (Plugins != null)
          foreach (Lazy<IPlugin, IPluginMetaData> objPlugin in Plugins)
            objColPlugins.Add(objPlugin.Value);
        // Devuelve la lista de plugins
        return objColPlugins;	
      }
    }
  }
}
```

Al ser la clase que vamos a utilizar como host, indicamos a **MEF** que queremos exportarla utilizando el atributo `Export` e indicando
que implementamos el interface `IPluginManager` que hemos definido al principio del artículo.
	
También nos definimos una variable global que va a almacenar nuestros plugins y la marcamos con otro atributo de **MEF**, en este
caso `ImportMany` indicando que vamos a importar no un objeto si no varios y de tipo `IPlugin` con metadatos de tipo 
`IPluginMetadata`.
	
El método que realiza la importación en este caso es `Initialize`. Existen varias formas de importar extensiones en **MEF**, la
más cómoda para mí es indicar una serie de directorios donde buscar nuestros ensamblados, para ello, a este método le pasamos
una cadena con directorios separados por punto y coma.
	
El método `Initialize` se crea un contenedor y le va añadiendo los ensamblados de los directorios. Automáticamente **MEF** nos
deja los plugins en la colección de plugins que definimos al principio (`objColPlugins`).
	
El resto de métodos únicamente nos sirven para llamar al método `InitLibraries` de cada uno de los plugins que tengamos cargado y
preparan una interface para que podamos acceder desde el resto de las clases de nuestra aplicación.
	
Nos queda por implementar la clase `HostController` que en teoría es la encargada de la comunicación entre las clases de plugins
y las ventanas de la aplicación. En nuestro caso sólo contiene el nombre de la aplicación y un método al que llaman los plugins
para mostrar un mensaje:
	
```csharp
// Controlador del host de Plugins
public class HostController: Bau.Libraries.LibPlugins.Core.Host.IHostController
{ 
	// Eventos públicos
	public event EventHandler<string> MessageAdded;

	public HostController()
	{ 
		ApplicationName = "TestPlugins";
	}
	
	// Muestra un mensaje
	public void ShowMessage(string strMessage)
	{ 
		if (MessageAdded != null)
			MessageAdded(this, strMessage);
	}
	
	// Nombre de la aplicación
	public string ApplicationName { get; }
}
```

Para terminar, he creado una ventana de WPF que presenta un cuadro de texto donde podemos introducir los directorios de los que vamos
a cargar los plugins y una lista con los plugins cargados.
	
El único método interesante de esta ventana es el que llama a la clase `PluginsManager` para cargar los plugins y añade un manejador
de eventos para recoger los mensajes enviado con el temporizador de la clase `PluginSampleHelloController` que vimos en la sección anterior:

```csharp
// Carga los plugins
private void LoadPlugins()
{	
	Plugins.HostController objHostController = new Plugins.HostController();
	Plugins.PluginsManager objPluginsManager = new Plugins.PluginsManager();
	string strError;

	// Inicializa el manager
	objPluginsManager.Initialize(typeof(MainWindow), txtPath.Text);
	// Inicializa los plugins
	objPluginsManager.InitPlugins(objHostController, out strError);
	// Añade los plugins a la lista
	lstPlugins.Items.Clear();
	foreach (IPlugin objPlugin in objPluginsManager.PluginsController)
		lstPlugins.Items.Add($"Plugin {objPlugin.Name} - {objPlugin.GetHelloMessage()}");
	// Añade el manejador de mensajes
	objHostController.MessageAdded += (objSender, strMessage) =>
		{ Dispatcher.Invoke(() => lstPlugins.Items.Add($"Mensaje recibido en el host: {strMessage}"),
													   System.Windows.Threading.DispatcherPriority.Normal);
		};
}
```

### Resumen

Es muy sencillo utilizar **MEF** para implementar extensiones en una aplicación. Como hemos visto, sólo tenemos que utilizar
una serie de atributos y cargar los plugins utilizando un contenedor.
	
Lo realmente complicado es definir lo que van a hacer nuestras extensiones y ampliar los interfaces de forma que sean lo más
flexibles posible.
	
Como siempre, el código fuente de las pruebas lo podeis encontrar en [GitHub](https://github.com/jbautistam/TestPlugins).