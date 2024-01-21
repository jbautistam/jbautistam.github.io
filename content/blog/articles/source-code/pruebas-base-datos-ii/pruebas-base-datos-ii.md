+++
title = "Pruebas de base de datos - Preparando el entorno"
date = "2019-10-11"
description = "A vueltas con las pruebas de base de datos: preparación del entorno, creación de bases de datos e inserción de datos de prueba"
thumbnail = "/articles/source-code/pruebas-base-datos/pruebas-base-datos.jpg"
tags = [ "Programación" ]
+++

Continuando con el [motor de pruebas de base de datos](/blog/articles/source-code/pruebas-base-datos/pruebas-base-datos)
del que hablábamos la semana pasada, hoy pasaremos a la creación de las bases de datos y dejaremos para
el próximo día la inserción de datos de pruebas.
 
Aunque el proyecto ha avanzado mucho, aún no tenemos código fuente en GitHub, quién sabe si la próxima semana sea la definitiva.

## Creación de bases de datos

Hoy explicaré cómo se crean las bases de datos que vamos a utilizar en estas pruebas.
El sistema en teoría debería funcionar con una base de datos semilla creada previamente y otra base de datos sobre
la que realizaremos las pruebas pero para que el ejemplo sea más completo, vamos a crear ambas bases de datos
al vuelo.
	
El archivo de conexiones que utilizaremos en esta ocasión es el siguiente:

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScriptsConnections>
	<Connection Key="Master" Type = "SqlServer">
		<Name>SqlServer - Master</Name>
		<Description/>
		<ConnectionString>Data Source=localhost;user=sa;password=MiTemporal#1P4ssw0rd;Initial Catalog=master</ConnectionString>
	</Connection>
	<Connection Key="Sales" Type = "SqlServer">
		<Name>SqlServer - Sales</Name>
		<Description/>
		<ConnectionString>Data Source=localhost;user=sa;password=MiTemporal#1P4ssw0rd;Initial Catalog=SalesDb</ConnectionString>
	</Connection>
	<Connection Key="Test" Type = "SqlServer">
		<Name>SqlServer - SalesForTest</Name>
		<Description/>
		<ConnectionString>Data Source=localhost;user=sa;password=MiTemporal#1P4ssw0rd;Initial Catalog=SalesDbTesting</ConnectionString>
	</Connection>
</DbScriptsConnections>
```

En ese archivo, tenemos tres conexiones: `Sales` es nuestra base de datos semilla (`SalesDb`) desde dónde recogeremos los datos
para las pruebas y `Test` es la base de datos dónde realizaremos las pruebas (`SalesDbTesting`). La tercera conexión `Master`
apunta a la base de datos `master` necesaria para poder ejecutar el script de creación de base de datos. Si las bases de datos
estuvieran creadas, no sería necesario pero dado que las bases de datos en teoría no existen, necesitamos poder conectarnos a
algo.

**Nota:** en realidad, podría trucar el cógio para conectarse a la base de datos master simplemente cambiando
en ejecución la cadena de conexión pero no quería liar el código con cosas como `if key == master`, al menos por ahora.

Una vez definidas las conexiones, veamos nuestro script de pasos para la creación:

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Name>Test copy data</Name>
	<PathProject>C:\Test\Scripts\CopyData</PathProject>

	<!-- Creación de la base de datos SalesDb -->
	<Step StartWithPreviousError = "false">
		<Name>Scripts database process: create database SalesDb</Name>
		<Script>DbScripts\01_Create_Database_Sales.xml</Script>
	</Step>

	<!-- Creación de la base de datos SalesDbTesting -->
	<Step StartWithPreviousError = "false">
		<Name>Scripts database process: create database SalesDbTesting</Name>
		<Script>DbScripts\03_Create_Database_SalesTest.xml</Script>
	</Step>
</DbScript>
```

Tenemos la ejecución de dos scripts: uno en teoría para crear la base de datos `SalesDb` y otro para la base
de datos `SalesDbTesting`.
	
**Nota:** Por ahora, he omitido los pasos para rellenar datos y copiar, los veremos más adelante.

El script de creación de la base de datos `SalesDb` (`01_Create_Database_Sales.xml`) contiene estas instrucciones:

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Block Name="Drop database if exists">
		<Execute Target ="Master">
			DROP DATABASE IF EXISTS [SalesDb];
		</Execute>
	</Block>
	
	<Block Name="Create database">
		<Execute Target ="Master">
			CREATE DATABASE [SalesDb]
			CONTAINMENT = NONE
			ON  PRIMARY
			( NAME = N'SalesDb', FILENAME = N'/var/opt/mssql/data/SalesDb.mdf', SIZE = 8192KB, MAXSIZE = UNLIMITED, FILEGROWTH = 65536KB )
			LOG ON
			( NAME = N'SalesDb_log', FILENAME = N'/var/opt/mssql/data/SalesDb_log.ldf', SIZE = 8192KB, MAXSIZE = 2048GB, FILEGROWTH = 65536KB );
		</Execute>
	</Block>

	<Block Name="Create tables">
		<Execute Target="Sales">
			ALTER DATABASE [SalesDb] SET ANSI_NULL_DEFAULT OFF;

			ALTER DATABASE [SalesDb] SET ANSI_NULLS OFF;

			ALTER DATABASE [SalesDb] SET ANSI_PADDING OFF;

			ALTER DATABASE [SalesDb] SET ANSI_WARNINGS OFF;

			ALTER DATABASE [SalesDb] SET ARITHABORT OFF;

			ALTER DATABASE [SalesDb] SET AUTO_CLOSE OFF;

			ALTER DATABASE [SalesDb] SET AUTO_SHRINK OFF;

			ALTER DATABASE [SalesDb] SET AUTO_UPDATE_STATISTICS ON;

			ALTER DATABASE [SalesDb] SET CURSOR_CLOSE_ON_COMMIT OFF;

			ALTER DATABASE [SalesDb] SET CURSOR_DEFAULT  GLOBAL;

			ALTER DATABASE [SalesDb] SET CONCAT_NULL_YIELDS_NULL OFF;

			ALTER DATABASE [SalesDb] SET NUMERIC_ROUNDABORT OFF;

			ALTER DATABASE [SalesDb] SET QUOTED_IDENTIFIER OFF;

			ALTER DATABASE [SalesDb] SET RECURSIVE_TRIGGERS OFF;

			ALTER DATABASE [SalesDb] SET  DISABLE_BROKER;

			ALTER DATABASE [SalesDb] SET AUTO_UPDATE_STATISTICS_ASYNC OFF;

			ALTER DATABASE [SalesDb] SET DATE_CORRELATION_OPTIMIZATION OFF;

			ALTER DATABASE [SalesDb] SET TRUSTWORTHY OFF;

			ALTER DATABASE [SalesDb] SET ALLOW_SNAPSHOT_ISOLATION OFF;

			ALTER DATABASE [SalesDb] SET PARAMETERIZATION SIMPLE;

			ALTER DATABASE [SalesDb] SET READ_COMMITTED_SNAPSHOT OFF;

			ALTER DATABASE [SalesDb] SET HONOR_BROKER_PRIORITY OFF;

			ALTER DATABASE [SalesDb] SET RECOVERY SIMPLE;

			ALTER DATABASE [SalesDb] SET  MULTI_USER;

			ALTER DATABASE [SalesDb] SET PAGE_VERIFY CHECKSUM;

			ALTER DATABASE [SalesDb] SET DB_CHAINING OFF;

			ALTER DATABASE [SalesDb] SET FILESTREAM( NON_TRANSACTED_ACCESS = OFF );

			ALTER DATABASE [SalesDb] SET TARGET_RECOVERY_TIME = 60 SECONDS;

			ALTER DATABASE [SalesDb] SET DELAYED_DURABILITY = DISABLED;

			EXEC sys.sp_db_vardecimal_storage_format N'SalesDb', N'ON';

			ALTER DATABASE [SalesDb] SET QUERY_STORE = OFF;

			SET ANSI_NULLS ON;

			SET QUOTED_IDENTIFIER ON;

			CREATE TABLE [dbo].[Products](
			[ProductId] [int] IDENTITY(1,1) NOT NULL,
			[Name] [varchar](50) NOT NULL,
			[Price] [float] NOT NULL,
			CONSTRAINT [PK_Products] PRIMARY KEY CLUSTERED
			(
			[ProductId] ASC
			)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
			) ON [PRIMARY];

			CREATE TABLE [dbo].[Sales](
			[SaleId] [int] IDENTITY(1,1) NOT NULL,
			[StoreId] [int] NOT NULL,
			[ProductId] [int] NOT NULL,
			[Date] [datetime] NOT NULL,
			[Units] [int] NOT NULL,
			[Price] [float] NOT NULL,
			CONSTRAINT [PK_Sales] PRIMARY KEY CLUSTERED
			(
			[SaleId] ASC
			)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
			) ON [PRIMARY];

			CREATE TABLE [dbo].[Stores](
			[StoreId] [int] IDENTITY(1,1) NOT NULL,
			[Name] [varchar](200) NOT NULL,
			CONSTRAINT [PK_Stores] PRIMARY KEY CLUSTERED
			(
			[StoreId] ASC
			)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
			) ON [PRIMARY];

			ALTER TABLE [dbo].[Sales]  WITH CHECK ADD  CONSTRAINT [FK_Sales_Products] FOREIGN KEY([ProductId])
			REFERENCES [dbo].[Products] ([ProductId])
			ON DELETE CASCADE;

			ALTER TABLE [dbo].[Sales] CHECK CONSTRAINT [FK_Sales_Products];

			ALTER TABLE [dbo].[Sales]  WITH CHECK ADD  CONSTRAINT [FK_Sales_Stores] FOREIGN KEY([StoreId])
			REFERENCES [dbo].[Stores] ([StoreId])
			ON DELETE CASCADE;

			ALTER TABLE [dbo].[Sales] CHECK CONSTRAINT [FK_Sales_Stores];

			ALTER DATABASE [SalesDb] SET  READ_WRITE;
		</Execute>
	</Block>
</DbScript>
```

Obviamente, ha salido del comando *Generar script de base de datos* de Sql Server Management Studio, lo único
que he añadido es el borrado de base de datos si existía previamente.
	
Los dos primeros comandos `Execute` van sobre la conexión `Master` para borrar la base
	de datos y crearla, mientras que el tercer comando `Execute` de creación de tablas, claves y relaciones 
	va sobre la conexión `Sales`
	
Para mantenerlo simple, sólo hemos creado tres tablas: Stores, Products y Sales.

**Nota:** eliminar la base de datos puede dar un error si la base de datos estaba creada y había
una conexión abierta. Existen formas de eliminar las conexiones antes de eliminar la base de datos pero para no
complicar el script, he utilizado la forma sencilla, es decir: si da errores, abrid vuestro servidor
y eliminad la base de datos a mano.
	
Ahora nos queda por ver el script de creación de nuestra base de datos de prueba `SalesDbTesting` (`03_Create_Database_SalesTest.xml`):

```XML
<DbScript>
	<Block Name="Drop database if exists">
		<Execute Target ="Master">
			DROP DATABASE IF EXISTS [SalesDbTesting];
		</Execute>
	</Block>
	
	<Block Name="Create database">
		<Execute Target ="Master">
			CREATE DATABASE [SalesDbTesting]
			CONTAINMENT = NONE
			ON  PRIMARY
			( NAME = N'SalesDbTesting', FILENAME = N'/var/opt/mssql/data/SalesDbTesting.mdf', SIZE = 8192KB, MAXSIZE = UNLIMITED, FILEGROWTH = 65536KB )
			LOG ON
			( NAME = N'SalesDbTesting_log', FILENAME = N'/var/opt/mssql/data/SalesDbTesting_log.ldf', SIZE = 8192KB, MAXSIZE = 2048GB, FILEGROWTH = 65536KB );
		</Execute>
	</Block>

	<Block Name="Create tables">
		<Execute Target="Test">
			ALTER DATABASE [SalesDbTesting] SET ANSI_NULL_DEFAULT OFF;

			...

			CREATE TABLE [dbo].[Products](
			[ProductId] [int] IDENTITY(1,1) NOT NULL,
			[Name] [varchar](50) NOT NULL,
			[Price] [float] NOT NULL,
			CONSTRAINT [PK_Products] PRIMARY KEY CLUSTERED
			(
			[ProductId] ASC
			)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
			) ON [PRIMARY];

           ...
		</Execute>
	</Block>
</DbScript>
```

He eliminado gran parte del script porque excepto la creación y las claves de conexiones, son exactamente iguales. Al fin y al cabo
vamos a hacer pruebas con bases de datos con la misma estructura así que en teoría, deberían tener el mismo esquema.

## Mejora en la creación de las bases de datos

No sé a vosotros, pero a mí me pone muy nervioso tener dos scripts exactamente iguales en los que únicamente cambian los nombres. 
Así que vamos a refactorizar nuestros scripts para que acepten variables. En un alarde de originalidad,
se llama `01_Create_Database_SalesVariable.xml`:
	
```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Block Name="Drop database if exists">
		<Execute Target ="Master">
			DROP DATABASE IF EXISTS [{{DbName:WithoutQuote}}];
		</Execute>
	</Block>

	<Block Name="Create database">
		<Execute Target ="Master">
			CREATE DATABASE [{{DbName:WithoutQuote}}]
			CONTAINMENT = NONE
			ON  PRIMARY
			( NAME = N'{{DbName:WithoutQuote}}', FILENAME = N'/var/opt/mssql/data/{{DbName:WithoutQuote}}.mdf', SIZE = 8192KB, MAXSIZE = UNLIMITED, FILEGROWTH = 65536KB )
			LOG ON
			( NAME = N'{{DbName:WithoutQuote}}_log', FILENAME = N'/var/opt/mssql/data/{{DbName:WithoutQuote}}_log.ldf', SIZE = 8192KB, MAXSIZE = 2048GB, FILEGROWTH = 65536KB );
		</Execute>
	</Block>

	<Block Name="Create tables">
		<Execute Target="{{ConnectionName}}">
		
			...
			
			ALTER DATABASE [{{DbName:WithoutQuote}}] SET QUERY_STORE = OFF;

			SET ANSI_NULLS ON;

			SET QUOTED_IDENTIFIER ON;

			CREATE TABLE [dbo].[Products](
			[ProductId] [int] IDENTITY(1,1) NOT NULL,
			[Name] [varchar](50) NOT NULL,
			[Price] [float] NOT NULL,
			CONSTRAINT [PK_Products] PRIMARY KEY CLUSTERED
			(
			[ProductId] ASC
			)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
			) ON [PRIMARY];

           ...

			ALTER DATABASE [{{DbName:WithoutQuote}}] SET  READ_WRITE;
		</Execute>
	</Block>
</DbScript>
```	

La aplicación que ejecuta los scripts, considera como una variable o expresión, las cadenas que están entre dobles llaves, es decir,
en nuestro script tenemos dos variables `ConnectionName` y `DbName`. La primera de ellas identifica la clave
de la conexión que se va a utilizar y `DbName` es la variable con el nombre de base de datos.
	
Lo normal es que el motor, sustituya el nombre de variable o la expresión por el valor calculado en tiempo de ejecución. Para que
sea más fácil, las fechas y las cadenas se ponen siempre entre apóstrofes (lo veremos en detalle en los siguientes artículos pero
sirva como explicación inicial). 

Sin embargo, en nuestro código tenemos sentencias de este estilo: `DbName:WithoutQuote`. Como decíamos, lo que hay
entre llaves es el nombre de variable pero en este caso se le aplica una máscara (lo que sigue a los dos puntos):
`WithoutQuote`. Esta máscara indica que en las cadenas no se pongan comillas, así evitamos tener errores 
en tiempo de ejecución como éste:
	
```SQL
FILENAME = N'/var/opt/mssql/data/{{DbName}}.mdf'
```	
	
Esta instrucción sustituye `DbName` por el valor de la variable entre apóstrofes que nos daría este resultado: 
	
```SQL
FILENAME = N'/var/opt/mssql/data/'Sales'.mdf'
```	
		
que por supuesto provoca un error en SQL Server, por eso, en su lugar utilizamos la máscara `WithoutQuote`:
	
```SQL
FILENAME = N'/var/opt/mssql/data/{{DbName:WithoutQuote}}.mdf'
```	
	
que nos devuelve este resultado: 
	
```SQL
FILENAME = N'/var/opt/mssql/data/Sales.mdf'
```	

que ya sería correcto.

Lo único que nos falta entonces es ver cómo se modifica el script de ejecución de pasos para añadir las variables:

```XML
<?xml version="1.0" encoding="utf-8"?>
<DbScript>
	<Name>Test copy data</Name>
	<PathProject>C:\Test\Scripts\CopyData</PathProject>

	<!-- Creación de la base de datos SalesDb -->
	<Step StartWithPreviousError = "false">
		<Name>Scripts database process: create database SalesDb</Name>
		<Script>DbScripts\01_Create_Database_SalesVariable.xml</Script>
		<Parameter Key="ConnectionName" Type="string" Value="Sales" />
		<Parameter Key="DbName" Type="string" Value="SalesDb" />
	</Step>

	<!-- Creación de la base de datos SalesDbTesting -->
	<Step StartWithPreviousError = "false">
		<Name>Scripts database process: create database SalesDbTesting</Name>
		<Script>DbScripts\01_Create_Database_SalesVariable.xml</Script>
		<Parameter Key="ConnectionName" Type="string" Value="Test" />
		<Parameter Key="DbName" Type="string" Value="SalesDbTesting" />
	</Step>
</DbScript>
```

Ahora llamamos dos veces a ejecutar el script `01_Create_Database_SalesVariable.xml`, con dos variables, `ConnectionName`
con la clave de conexión y `DbName` con el nombre de base de datos y así, simplemente cambiando los valores de estas dos 
variables podemos utilizar el mismo script para crear dos bases de datos diferentes.

## Conclusión

Por ahora eso es todo. En el [siguiente artículo](/blog/articles/source-code/pruebas-base-datos-iii/pruebas-base-datos-iii)
continuaré con los scripts de creación de datos de prueba y de copia de datos y veremos algunas
sentencias un poco más complejas como los bucles.