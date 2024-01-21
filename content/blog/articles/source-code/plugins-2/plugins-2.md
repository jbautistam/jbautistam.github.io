+++
title = "Desarrollo de plugins con MEF"
date = "2019-10-11"
description = "Desarrollo de plugins para aplicaciones .Net utilizando MEF"
thumbnail = "/articles/source-code/plugins-2/plugins-2.jpg"
tags = [ "Programación" ]
+++

Hace ya algunos años, escribía sobre la [creación de plugins](/blog/articles/source-code/plugins/plugins) 
para aplicaciones.NET utilizando la primera versión de MEF.
	
Por recordarlo, un plugin es una aplicación o librería a la que podemos acceder desde nuestra aplicación
sin necesidad de enlazarla en tiempo de compilación. Es decir, no tenemos
que añadir una referencia en nuestro proyecto, al ejecutar nuestra aplicación se carga la dependencia externa
y se realizan las llamadas oportunas.
	
A muchos les sonará como inyección de dependencias y en cierto modo es así, la única diferencia real
es que en ninguna parte de nuestro proyecto o solución necesitamos una referencia a la librería que
implementa las interfaces.
	
En esta ocasión vamos a utilizar la última versión de MEF (en algún momento se llamó MEF 2 pero finalmente
Microsoft ha optado por eliminar el número de versión).
	
Ha habido algún cambio importante respecto a la primera versión que describía en el artículo anterior, quizá el más importante es que se ha eliminado
la necesidad de incluir atributos `[Export]` en los plugins indicando que cumplían con un interface. Además
ha desaparecido el método para cargar ensamblados de un directorio y ahora hay que añadirlos manualmente al
contenedor, pero en general, se ha mejorado bastante la forma de trabajar con los componentes.

## El interface de los plugins en el host

Para trabajar con plugins, vamos a utilizar dos proyectos, el primero de ellos, para entendernos, lo vamos a llamar
host: este proyecto es el ejecutable o ensamblado principal de nuestra aplicación y será el encargado de lanzar
los plugins.

Para comenzar, debemos incluir en nuestro proyecto de Host el paquete de Nuget 
[System.Composition](https://www.nuget.org/packages/System.Composition/) que es el que nos ofrece las funcionalidades de MEF.
	
En segundo lugar, necesitamos un interface que es el que deben implementar los plugins. Por supuesto, el interface lo
podemos definir como cualquier otro interface. Para el ejemplo, he decidido crear una consola que lance procesadores
de comandos o pasos de trabajo, por eso he definido el siguiente interface para los plugins:
	
```csharp
using System;
using System.Threading.Tasks;

using Bau.Libraries.LibJob.Application.Models.Processes;

namespace Bau.Libraries.LibJob.Application.Interfaces
{
	public interface IJobStepProcessor
	{
		void Initialize(Models.Context.JobContextModel context, ProcessStepModel step);

		Task<bool> ProcessAsync();

		string Key { get; }
	}
}
```	

Lo que pretendo es que haya diferentes procesadores de comandos a los que se les envíe la información de contexto (donde
se podrían incluir cadenas de conexión, directorios, nombres de archivos...) y los datos del paso (dónde estaría
por ejemplo el nombre del archivo de script y sus parámetros, por ejemplo). 
	
Por supuesto, el plugin también debe tener acceso a las definiciones de estos objetos que estarían en una librería de
módulos comunes. Por no hacer más largo el ejemplo, no vamos a ver estas definiciones pero las podéis encontrar en
el código fuente en [GitHub](https://github.com/jbautistam/JobManager).

## El proyecto de plugin

Nuestro proyecto de plugin, para este caso, el procesador de pasos, debe incluir también el paquete 
[System.Composition](https://www.nuget.org/packages/System.Composition/) e implementar la interface que hemos definido en el host.
	
Para que sea más sencillo, suelo definir en la librería común donde se encuentra la interface de los plugins
una clase base abstracta encargada de la 'fontanería' de logging, propiedades y demás
aunque no es estrictamente necesario:
	
```csharp
using System;
using System.Threading.Tasks;

using Bau.Libraries.LibJob.Application.Models.Processes;

namespace Bau.Libraries.LibJob.Application.Processor
{
	public abstract class JobStepProcessorBase: Interfaces.IJobStepProcessor
	{
		public JobStepProcessorBase(string key)
		{
			Key = key;
		}

		public void Initialize(Models.Context.JobContextModel context, ProcessStepModel step)
		{
			Context = context;
			Step = step;
		}

		public async Task<bool> ProcessAsync()
		{
			Context.UpdateJobStep(Key, Step);
			Context.WriteDebug("Start process");
			await ExecuteStepAsync();
			Context.WriteDebug("End process");
			return Context.Errors.Count == 0;
		}
		
		protected abstract Task ExecuteStepAsync();

		public string Key { get; }

		public ProcessStepModel Step { get; private set; }

		protected Models.Context.JobContextModel Context { get; private set; }
	}
}
```
	
Y en la librería de plugins, nos quedaría por tanto heredar de la clase abstracta e implementar el método `ExecuteStepAsync`.

Para que el ejemplo sea sencillo: el plugin no ejecuta ningún proceso 'práctico', simplemente lee un archivo de texto 
donde en teoría están las instrucciones del paso, sustituye ciertos marcadores en el texto por los argumentos pasados 
desde el host y lo muestra en la consola:
	
```csharp
using System;
using System.Threading.Tasks;

using Bau.Libraries.LibCommonHelper.Extensors;

namespace Bau.Libraries.LibJob.Processor.Test
{
	public class TestProcessor: Application.Processor.JobStepProcessorBase
	{
		public TestProcessor(): base("TestProcessor") {}

		protected async override Task ExecuteStepAsync()
		{
			if (!System.IO.File.Exists(Step.ScriptFileName))
				Context.WriteError($"Can't find the file {Step.ScriptFileName}");
			else
			{
				string text = System.IO.File.ReadAllText(Step.ScriptFileName);

					text = ReplaceParameters(text);
					Context.WriteInfo(text);
			}
			await Task.Delay(100);
		}

		private string ReplaceParameters(string text)
		{
			foreach ((string key, object value) in Step.GetCombinedParameters().Enumerate())
				if (value == null)
					text = text.ReplaceWithStringComparison("{{" + key + "}}", "NULL");
				else
					text = text.ReplaceWithStringComparison("{{" + key + "}}", value.ToString());
			return text;
		}
	}
}
```

## La conexión del host con el plugin

Hasta ahora, tenemos dos librerías separadas, una de host y otra de plugin, pero no hemos conectado 
ambas aún. De alguna forma debemos indicar en el host que se carguen los plugins en el contenedor de MEF.
	
Hay varias formas de hacerlo que se pueden ver en la documentación. Para mí, la más sencilla es generar los plugins
en directorios y añadir al contenedor los ensamblados de esos directorios que cumplen con la interface. Veamos el
código de carga de plugins:
	
```csharp
using System;
using System.Collections.Generic;
using System.Composition;
using System.Composition.Hosting;
using System.Linq;
using System.Reflection;

namespace Bau.Libraries.LibJob.Application.Processor.Controllers
{
	internal class PluginsManager<TypePlugin>: IDisposable
	{
		public void Initialize(List<string> pathPlugins, string extensionPlugins = ".plugin.dll")
		{
			Container = new ContainerConfiguration().WithAssemblies(GetPluginsAssemblies(pathPlugins, NormalizeExtension(extensionPlugins)), 
																	GetConventionsBuilder()).CreateContainer();
			Plugins = Container.GetExports<TypePlugin>().ToList();
		}

		private string NormalizeExtension(string extension)
		{
			if (string.IsNullOrWhiteSpace(extension))
				extension = ".plugin.dll";
			extension = extension.Trim();
			if (!extension.StartsWith("."))
				extension = "." + extension;
			return extension;
		}

		private IEnumerable<Assembly> GetPluginsAssemblies(List<string> pathPlugins, string extensionPlugins)
		{
			List<Assembly> assemblies = new List<Assembly>();

				foreach (string path in pathPlugins)
					if (System.IO.Directory.Exists(path))
						foreach (string fileName in System.IO.Directory.GetFiles(path, "*" + extensionPlugins))
							if (System.IO.File.Exists(fileName) && fileName.EndsWith(extensionPlugins, StringComparison.CurrentCultureIgnoreCase))
								try
								{
									assemblies.Add(Assembly.LoadFrom(fileName));
								}
								catch (Exception exception)
								{
									Errors.Add($"Error when load plugin {fileName}{Environment.NewLine}{exception.Message}");
								}
				return assemblies;
		}

		private System.Composition.Convention.ConventionBuilder GetConventionsBuilder()
		{
			System.Composition.Convention.ConventionBuilder builder = new System.Composition.Convention.ConventionBuilder();

				builder.ForTypesDerivedFrom<TypePlugin>().Export<TypePlugin>().Shared();
				return builder;
		}

		protected virtual void Dispose(bool disposing)
		{
			if (!Disposed)
			{
				if (disposing)
					Container = null;
				Disposed = true;
			}
		}

		public void Dispose()
		{
			Dispose(true);
		}

		[ImportMany]
		public IEnumerable<TypePlugin> Plugins { get; private set; }

		private CompositionContext Container { get; set; }

		public List<string> Errors { get; } = new List<string>();

		public bool Disposed { get; private set; }
	}
}
```
	
Sí, un poco largo, lo sé, pero en realidad lo importante es que la clase recibe una lista de directorios donde en teoría
se encuentran los plugins y los añade a un contenedor utilizando la siguiente instrucción:

```csharp
Container = new ContainerConfiguration().WithAssemblies(GetPluginsAssemblies(pathPlugins, NormalizeExtension(extensionPlugins)), 
					GetConventionsBuilder()).CreateContainer();
```

El método `GetPluginsAssemblies` lee los ensamblados de los diferentes directorios y los añade a una lista
de tipo `Assembly`. La variable `extensionPlugins` identifica la extensión de los archivos de plugins, puede ser
simplemente `.dll` pero en caso de proyectos con muchos ensamblados, quizá sea mejor marcar el ensamblado principal
(el que cumple con la interface de plugin) con una extensión particular como por ejemplo `.plugin.dll` (de todas
formas, esto no es obligatorio).
	
Por su parte, el método `GetConventionsBuilder()` se asegura que sólo cargamos en el contenedor los ensamblados
que cumplen con la interface definida en el genérico `TypePlugin`.
	
**Nota:** Para los puristas, en nuestro caso, no implementa directamente la interface
si no que hereda de una clase abstracta que implementa la interface. Funciona igualmente.
	
Cuando finaliza la carga, en la propiedad `Plugins` vamos a tener cargados todos los plugins que hemos obtenido
del contenedor con esta instrucción del método `Initialize`:
	
```csharp
Plugins = Container.GetExports<TypePlugin>().ToList();
```	

## Ejecución de los plugins

Por último, lo que nos queda es aprovechar los plugins que tenemos cargados y ejecutarlos.

En el caso de ejemplo, nuestra aplicación principal, carga los plugins y los inserta en una lista de procesadores:
	
```csharp
Processor.Controllers.PluginsManager<Processor.JobStepProcessorBase> pluginsManager = new Processor.Controllers.PluginsManager<Processor.JobStepProcessorBase>();

pluginsManager.Initialize(configuration.PathPlugins, ".dll");
if (pluginsManager.Errors.Count == 0)
{
	foreach (Processor.JobStepProcessorBase processor in pluginsManager.Plugins)
		Processors.Add(processor.Key, processor);
	initialized = true;
}
else
	foreach (string pluginError in pluginsManager.Errors)
		Context.Logger.WriteError("JobManager - Initialize plugins", pluginError);
```

Y llama al procesador adecuado para la ejecución de cada paso:

```csharp
private async Task ProcessStepAsync(ProcessStepModel step)
{
	Processor.JobStepProcessorBase processor = Processors[step.PluginKey];

		if (processor == null)
			Context.WriteError("JobManager - Process", $"Can't find the processor for step {step.PluginKey}");
		else
		{
			try
			{
				processor.Initialize(Context, step);
				await processor.ProcessAsync();
			}
			catch (Exception exception)
			{
				Context.WriteError("JobManager - Process", exception.Message);
			}
		}
}
```

Una nota importante que me dio algunos problemas al principio: si tenemos el código fuente del host y los plugins en una
solución de Visual Studio, lo podemos ejecutar y depurar sin problemas pero cuando hagamos modificaciones debemos
recompilar toda la solución porque los plugins no se compilan siempre cuando depuramos. 
	
Supongo que es porque la optimización de la compilación busca las referencias de proyectos
en la librería que contiene el ejecutable pero como en realidad no hay ninguna referencia a los plugins, no los compila
de nuevo. Por eso hay que hacerlo a mano o modificar la configuración del compilador para que siempre compile toda la
solución. Yo acabé asociando la tecla F7 al comando 'Recompilar toda la solución' pero va en gustos.

Y eso es todo, simple y limpio.

Como siempre, podéis ver el código fuente en [GitHub](https://github.com/jbautistam/JobManager).