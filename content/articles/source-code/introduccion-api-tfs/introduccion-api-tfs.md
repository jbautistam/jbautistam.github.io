+++
title = "Introducción a la API de TFS"
date = "2019-10-11"
description = "Una breve introducción al API de TFS con C#"
thumbnail = "/articles/source-code/introduccion-api-tfs/introduccion-api-tfs.jpg"
tags = [ "Programación" ]
+++

Estos días he estado jugando con el acceso a **Team Foundation Server** utilizando sus API y servicios
Web.
	
Aunque una vez que conoces sus bases es bastante fácil de utilizar, la documentación es, en el mejor de los casos, escasa
y dispersa por eso he decidido escribir una breve introducción a **TFS** para programadores.
	
Por supuesto, antes de comenzar debemos utilizar **NuGet** para descargar y añadir las referencia al API. En mi caso
he añadido las referencias a:
	
* `Microsoft.TeamFoundationServer.Client`
* `Microsoft.TeamFoundationServer.ExtendedClient`

Aunque las APIs de **Team Foundation Server** nos ofrecen acceso en teoría a todos los módulos de TFS, a mí me interesaba
particularmente el acceso a los proyectos y tareas, por eso este artículo está relacionado con estos elementos.	

## Conexión a TFS

En primer lugar, necesitamos definir el objeto que nos va a dar el acceso a los diferentes apartados de la API, 
he definido esta variable global en la clase que almacena mis métodos de acceso a TFS:
	
```csharp
private TfsTeamProjectCollection tfsTeamProject = null;
```

Lo primero que debemos hacer por supuesto es conectarnos con el servidor de **Team Foundation Server**, para
ello utilizo este código:

```csharp
public void Connect(string strUrl, string strLogin, string strPassword)
{ 
	WindowsCredential objWindowsCredentials = new WindowsCredential(new NetworkCredential(strLogin, strPasword));
  	TfsClientCredentials objCredentials = new TfsClientCredentials(objWindowsCredentials);

	// Crea una conexión
	tfsTeamProject = new TfsTeamProjectCollection(strUrl, objCredentials);
	// Autentifica
	tfsTeamProject.Authenticate();
}
```

La URL del servicio de Team Foundation Server se compone de la URL del servidor TFS al que estamos conectando y 
la colección a la que queremos conectar. En mis experimentos he utilizado la versión online de TFS, es decir, la
cuenta gratuita que obtuve en la
[web de Visual Studio](https://www.visualstudio.com/en-us/products/visual-studio-team-services-vs.aspx).
	
Esta Web me da una Url de acceso del tipo `https://usuario.visualstudio.com`. Como colección utilizo la
predeterminada `DefaultCollection`. Así, la Url que le pasamos al método debe ser en este caso:
`https://usuario.visualstudio.com/DefaultCollection`.
	
Por supuesto, el login y la contraseña son los que nos ha dado el sistema inicialmente.

## Proyectos en TFS

Para cargar los proyectos que tenemos en TFS, una vez abierta la conexión utilizo un método similar a este:

```csharp
WorkItemStore objWorkItemStore = tfsTeamProject.GetService<WorkItemStore>();

// Carga los proyectos
foreach (Project objTfsProject in objWorkItemStore.Projects)
 .... tratamiento del proyecto de TFS
```

Como se puede ver en el código fuente en **GitHub**, he definido una serie de clases de modelo 
(como `ProjectModel`) que mantienen los datos en memoria para
no tener que acceder continuamente al servidor. Estas clases de modelo son bastante sencillas. Es decir: propiedades
con el nombre, la descripción, el Id y colecciones de elementos relacionados (en el caso de los proyectos tendríamos
por ejemplo la colección de tareas):

```csharp
/// < summary>
///	Obtiene los proyectos
/// < /summary>
private ProjectModelCollection GetProjects()
{ 
	ProjectModelCollection objColProjects = new ProjectModelCollection();
  	WorkItemStore objWorkItemStore = tfsTeamProject.GetService<WorkItemStore>();

	// Carga los proyectos
	foreach (Project objTfsProject in objWorkItemStore.Projects)
	{ 
		ProjectModel objNewProject = new ProjectModel();
		
		// Asigna los datos
		objNewProject.ID = objTfsProject.Id;
		objNewProject.Name = objTfsProject.Name;
		// Asigna las categorías
		objNewProject.Categories.AddRange(LoadCategories(objTfsProject));
		// Añade los tipos de tareas
		for (int intIndex = 0; intIndex < objTfsProject.WorkItemTypes.Count; intIndex++)
			objNewProject.TaskTypes.Add(objTfsProject.WorkItemTypes[intIndex].Name);
		// Recorre las iteraciones
		for (int intIndex = 0; intIndex < objTfsProject.IterationRootNodes.Count; intIndex++)
		{ 
			NodeModel objNode;
			IterationModel objIteration = new IterationModel();
			
			// Añade el nodo
			objNode = objNewProject.Iterations.Add(null, objTfsProject.IterationRootNodes[intIndex].Id,
			                               objTfsProject.IterationRootNodes[intIndex].Name, objIteration);
			// Asigna los objetos Lazy
			objIteration.LazyTasks.LazyData = new Lazy<TaskModelCollection>(() => LoadTasksIteration(objNewProject, objNode.ID?? 0));
		}
		// Añade los objetos Lazy
		objNewProject.LazyTasks.LazyData = new Lazy<TaskModelCollection>(() => LoadTasks(objNewProject));
		// Y añade el proyecto a la colección
		objColProjects.Add(objNewProject);
	}
	// Devuelve los proyectos
	return objColProjects;
}
```

Si nos fijamos, la clase de proyecto de TFS (`Project`) nos da acceso no solo a los datos del proyecto si no también a
sus categorías, tipos de tarea e iteraciones (entre otros).
	
Para cargar las tareas, utilizo un objeto de tipo `Lazy` es decir, no se cargarán las tareas hasta que no se vayan a
mostrar, esto permite mejorar el rendimiento de la aplicación puesto que no hacemos la llamada al servidor hasta que
no es realmente necesario.

## Tareas en TFS

La carga de tareas en TFS es ligeramente más truculenta porque se realiza a partir de consultas, no existe
un método del proyecto que nos descargue todas las tareas de un proyecto si no que tenemos que realizar la consulta
adecuada. Existen formas para almacenar consultas a partir de objetos del mismo modo que lo hace **Visual Studio**
pero para mi aplicación he utilizado el acceso por consultas SQL en formato de texto.
	
Como ejemplo, aquí dejo el código para cargar todas las tareas de un proyecto y de una iteración:

```csharp
/// < summary>
///	Obtiene los WorkItems
/// < /summary>
private TaskModelCollection LoadTasks(ProjectModel objProject)
{ 
	return QueryWorkItems(objProject, $"SELECT * FROM WorkItems WHERE [System.TeamProject] = '{objProject.Name}'");
}

/// < summary>
///	Carga las tareas de una iteración
/// < /summary>
private TaskModelCollection LoadTasksIteration(ProjectModel objProject, int intIDIteration)
{ 
	return QueryWorkItems(objProject, $"SELECT * FROM WorkItems WHERE [System.TeamProject] = '{objProject.Name}' AND [System.IterationId] = '{intIDIteration}'");
}

/// < summary>
///	Consulta los elementos de trabajo y los transforma en una colección de tareas
/// < /summary>
private TaskModelCollection QueryWorkItems(ProjectModel objProject, string strQuery)
{ 
	Project objTfsProject = LoadProjectTfs(objProject);
	TaskModelCollection objColTasks = new TaskModelCollection();
	
	// Carga las tareas del proyecto
	if (objTfsProject != null)
	{ 
		WorkItemStore objWorkItemStore = tfsTeamProject.GetService<WorkItemStore>();
		WorkItemCollection objColWorkItems = objWorkItemStore.Query(strQuery);
		
		// Transforma los elementos de trabajo en tareas
		foreach (WorkItem objWorkItem in objColWorkItems)
		{ 
			TaskModel objTask = new TaskModel();
			
			  // Asigna los datos
			  objTask.ID = objWorkItem.Id;
			  objTask.Name = objWorkItem.Title;
			  objTask.Description = objWorkItem.Description;
			  objTask.IdIteration = objWorkItem.IterationId;
			  objTask.Project = objWorkItem.Project.Name;
			  objTask.AssignedTo = GetWorkItemField(objWorkItem, CoreField.AssignedTo);
			  objTask.DateUpdate = objWorkItem.ChangedDate;
			  objTask.UserUpdate = objWorkItem.ChangedBy;
			  objTask.DateNew = objWorkItem.CreatedDate;
			  objTask.UserNew = objWorkItem.CreatedBy;
			  objTask.State = objWorkItem.State;
			  objTask.Tags = objWorkItem.Tags;
			  objTask.Type = objWorkItem.Type.ToString();
			  objTask.DateAuthorized = objWorkItem.AuthorizedDate;
			  objTask.UserAuthorized = GetWorkItemField(objWorkItem, CoreField.AuthorizedAs);
			  objTask.BoardColumn = GetWorkItemField(objWorkItem, CoreField.BoardColumn);
			  objTask.BoardColumnDone = GetWorkItemField(objWorkItem, CoreField.BoardColumnDone);
			  objTask.BoardLane = GetWorkItemField(objWorkItem, CoreField.BoardLane);
			  objTask.LinkType =  GetWorkItemField(objWorkItem, CoreField.LinkType);
			  objTask.NodeName = objWorkItem.NodeName;
			  objTask.Reason = objWorkItem.Reason;
			  objTask.RelatedLinkCount = GetWorkItemField(objWorkItem, CoreField.RelatedLinkCount).GetInt(0);
			  // Añade la tarea a la colección
			  objColTasks.Add(objTask);
		}
	}
	// Devuelve la colección
	return objColTasks;
}

/// < summary>
///	Obtiene un campo de una consulta de elementos de trabajo
/// < /summary>
private string GetWorkItemField(WorkItem objWorkItem, CoreField intIDField)
{ 
	object objValue = null;
	
	// Obtiene el valor
	if (objWorkItem.Fields.Contains((int) intIDField))
		objValue = objWorkItem.Fields[intIDField].Value;
	// Convierte el valor a una cadena
	if (objValue == null)
		return null;
	else
		return objValue.ToString();
}
```

Debemos tener en cuenta que la carga de tareas nos da los `WorkItem` en una lista, es decir,
sabemos que los `WorkItem` pueden tener tareas asociadas pero TFS no las almacena ni las devuelve como 
un árbol si no que nos devuelve una lista de relaciones para cada tarea en la colección
`Links`. En en esta colección donde nos indica las relaciones entre las tareas, es decir, si una
tarea es hija de o padre de o se tiene que ejecutar antes o después de.

## Código fuente

Como siempre, el código fuente de la aplicación que estoy utilizando para pruebas se encuentra en 
[GitHub](https://github.com/jbautistam/TFSample).
	
La aplicación no está completa. No carga aún las relaciones entre tareas que describíamos en el punto anterior aunque
espero que sirva como base en vuestras propias pruebas con TFS.