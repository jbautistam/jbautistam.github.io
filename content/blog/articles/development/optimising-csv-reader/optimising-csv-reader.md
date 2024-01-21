+++
title = "Optimización de lectura de archivos CSV"
date = "2023-11-19"
description = "Optimización de lectura de archivos CSV"
thumbnail = "/articles/development/optimising-csv-reader/optimising-csv-reader.png"
tags = [ "Programación" ]
+++

Llevo ya algún tiempo con la curiosidad de aprovechar los tipos `Span` que aparecieron en versiones anteriores
de .Net.

Los tipos [Span](https://learn.microsoft.com/en-us/archive/msdn-magazine/2018/january/csharp-all-about-span-exploring-a-new-net-mainstay) 
y `ReadOnlySpan` en resumen, nos dan acceso a la memoria sobre objetos administrados de una forma mucho
más sencilla y están sobre todo pensadas para optimizar ciertos procesos como el manejo de cadenas.

Tenía una librería de lectura de archivos CSV en [GitHub](https://github.com/jbautistam/LibCsvFiles) que parecía la más
adecuada para este tipo de desarrollo dado que trabaja muchísimo con cadenas y creo que era el candidato idóneo para este tipo
de optimización.

La librería cumple con el formato de lectura de archivos CSV:

* Los campos se separan por una coma (aunque esto es configurable siempre que el separador sea un carácter).
* Los campos que contienen una coma, se rodean entre comillas.
* Las comillas que aparecen en un campo, se escapan utilizando otra comilla (es decir, `12""3`, se lee como `12"3`).

Además:

* Para separar los registros se utilizan saltos de línea de tipo `\r\n`.
* Están permitidos los saltos de línea dentro de un campo siempre que se rodeen por comillas.

La librería se divide en una clase de lectura de archivos y otra de escritura. En este caso me he centrado en la clase
de lectura.

La clase de lectura implementa el interface `IDataReader` sobre el archivo CSV.

Un ejemplo básico de utilización de la clase sería algo así:

```csharp
using (CsvReaderNotOptimized reader = new(null, null))
{
	object? value;

		// Abre el archivo
		reader.Open(GetFullFileName(FileName));
		// Recorre todos los registros
		while (reader.Read())
			for (int index = 0; index < reader.FieldCount; index++)
				value = reader.GetValue(index);
}
```

## Pruebas unitarias

Por supuesto, cuando se va a realizar cualquier trabajo de optimización, lo mejor es comenzar escribiendo pruebas unitarias. 
Vamos a hacer cambios en el código, debemos asegurarnos que todos los cambios que hagamos no provoquen errores
en la lectura de los archivos.

Para las pruebas unitarias, creé una serie de archivos CSV sencillos con los casos que quería comprobar (tipos de datos, comillas, saltos
de línea, nulos...).

Por cada uno de los archivos CSV hay un archivo de esquema que contiene los nombres de los campos y la definición de archivos (carácter
de los separadores de campos, carácter decimal, formato de las fechas...).

Con esos dos archivos, generé una clase [JsonGenerator](https://github.com/jbautistam/LibCsvFiles/blob/main/tests/LibCsvFilesTests/Generator/JsonGenerator.cs)
que crea un archivo JSON con los datos leidos de un archivo CSV.

Por supuesto, como el archivo JSON se generan a partir del lector de CSVs que queremos probar, debemos asegurarnos que los archivos de salida
de este generador son totalmente correctos.

La clase `JsonGenerator` contiene un método `generate_all_response_files` que realmente es un método de prueba (está marcado
con un atributo `[Fact]` de [xUnit](https://xunit.net/)) aunque está marcado con el atributo `Skip` para que no se ejecute cada vez que ejecutemos
las pruebas.

Podríamos haber dejado este generador en otro ejecutable fuera del proyecto de pruebas pero al tenerlo en el mismo proyecto, cuando queremos
regenerar los archivos simplemente debemos eliminar el atributo `Skip` y ejecutar las pruebas. Es cómodo.

Las pruebas unitaria realmente se encuentran en la clase 
[CsvReader_should](https://github.com/jbautistam/LibCsvFiles/blob/main/tests/LibCsvFilesTests/CsvReader_should.cs), este método
recorre todos los archivos CSV del directorio `Data` y compara la salida del `IDataReader` con el contenido de los archivos
JSON.

## El punto de partida

Antes de comenzar a optimizar, copié la clase `CsvReader` sobre `CsvReaderNotOptimized`, que posiblemente antes o después
eliminaré del código fuente en GitHub. Para poder comparar con la versión final, el código es este:

```csharp
using System.Data;

using Bau.Libraries.LibCsvFiles.Extensors;
using Bau.Libraries.LibCsvFiles.Models;

namespace Bau.Libraries.LibCsvFiles;

/// <summary>
///		Implementación de <see cref="IDataReader"/> para archivos CSV
/// </summary>
public class CsvReaderNotOptimized : IDataReader
{
	// Eventos públicos
	public event EventHandler<EventArguments.AffectedEvntArgs>? ReadBlock;
	// Variables privadas
	private bool _streamOpenedFromReader;
	private System.IO.StreamReader? _fileReader;
	private List<object?>? _recordsValues;
	private long _row;

	public CsvReaderNotOptimized(FileModel? definition, List<ColumnModel>? columns, int notifyAfter = 10_000)
	{
		FileDefinition = definition ?? new FileModel();
		Columns = columns ?? new List<ColumnModel>();
		NotifyAfter = notifyAfter;
	}

	/// <summary>
	///		Abre el archivo
	/// </summary>
	public void Open(string fileName)
	{
		// Indica que el stream se ha abierto en la librería
		_streamOpenedFromReader = true;
		// Abre el archivo sobre el stream
		Open(new System.IO.StreamReader(fileName, true));
	}

	/// <summary>
	///		Abre el datareader sobre el stream
	/// </summary>
	public void Open(System.IO.StreamReader stream)
	{
		// Guarda el stream
		_fileReader = stream;
		// Lee las cabeceras
		ReadHeader();
		// e indica que aún no se ha leido ninguna línea
		_row = 0;
	}

	/// <summary>
	///		Interpreta la cabecera del archivo utilizando cabeceras de columnas tipadas. 
	///		Las cabeceras de columnas con el tipo, tienen la estructura Nombre|Tipo
	///		Sólo se interpreta la cabecera cuando no hay columnas y el archivo está marcado indicando que contiene columnas tipadas en la cabecera
	/// </summary>
	private void ReadHeader()
	{
		if (_fileReader is not null && !_fileReader.EndOfStream && FileDefinition.WithHeader)
		{
			string? line = _fileReader.ReadLine();

				// Interpreta las columnas (si no se han definido)
				if (Columns.Count == 0 && !string.IsNullOrEmpty(line))
					foreach (string field in ParseLine(line.Trim()))
						if (FileDefinition.TypedHeader)
						{
							string[] parts = field.Split('|');

								if (parts.Length == 2)
									Columns.Add(new ColumnModel
														{
															Name = parts[0],
															Type = parts[1].GetEnum(ColumnModel.ColumnType.String)
														}
												);
								else
									throw new NotImplementedException($"Can't extract the column type from header ({field})");
						}
						else
							Columns.Add(new ColumnModel
												{
													Name = field,
													Type = ColumnModel.ColumnType.String
												}
										);
		}
	}

	/// <summary>
	///		Lee un registro
	/// </summary>
	public bool Read()
	{
		bool readed = false;
		string line = ReadLine();

			// Interpreta los datos
			if (!string.IsNullOrWhiteSpace(line))
			{
				// Interpreta la línea
				_recordsValues = ConvertFields(ParseLine(line));
				// Incrementa el número de línea y lanza el evento
				_row++;
				RaiseEventReadBlock(_row);
				// Indica que se han leido datos
				readed = true;
			}
			else
				_recordsValues = null;
			// Devuelve el valor que indica si se han leído datos
			return readed;
	}

	/// <summary>
	///		Lee la siguiente línea no vacía del archivo
	/// </summary>
	private string ReadLine()
	{
		string line = string.Empty;
		bool mustReadNextLine = false;
		int quotes = 0;

			// Lee la siguiente línea no vacía y se salta las líneas de cabecera
			while (_fileReader != null && !_fileReader.EndOfStream && (string.IsNullOrWhiteSpace(line) || mustReadNextLine))
			{
				string? part = _fileReader.ReadLine();

					// Resetea el valor que indica que la siguiente vez se debe leer la siguiente línea
					mustReadNextLine = false;
					// Lee la línea
					// Cuenta el número de comillas (los acumula porque puede que haya saltos de línea intermedios que tengan también comillas)
					quotes += CountQuotes(part);
					// Se debe leer la siguiente línea si el número de caracteres de comillas no es par, eso quiere decir que ha habido un salto
					// de línea en un campo
					if (quotes % 2 != 0)
					{
						// Añade a la línea el salto de línea que ha borrado FileReader.ReadLine()
						part += Environment.NewLine;
						// Indica que se debe leer una línea más
						mustReadNextLine = true;
					}
					// Añade la sección a la línea
					line += part;
			}
			// Quita los espacios de la línea
			if (!string.IsNullOrWhiteSpace(line))
				line = line.Trim();
			// Devuelve la línea leida
			return line;
	}

	/// <summary>
	///		Cuenta el número de comillas que hay en una línea
	/// </summary>
	private int CountQuotes(string? line)
	{
		int number = 0;

			// Si hay algo que contar
			if (!string.IsNullOrWhiteSpace(line))
			{
				// Quita los caracteres de escape (\")
				line = line.Replace("\\\"", "");
				// Cuenta las comillas de la cadena
				foreach (char chr in line)
					if (chr == '"')
						number++;
			}
			// Devuelve el número contado
			return number;
	}

	/// <summary>
	///		Lanza el evento de lectura de un bloque
	/// </summary>
	private void RaiseEventReadBlock(long row)
	{
		if (NotifyAfter > 0 && row % NotifyAfter == 0)
			ReadBlock?.Invoke(this, new EventArguments.AffectedEvntArgs(row));
	}

	/// <summary>
	///		Interpreta la línea separando los campos teniendo en cuenta las comillas
	/// </summary>
	private List<string> ParseLine(string line)
	{
		List<string> fields = new List<string>();
		string field = string.Empty;
		bool isInQuotes = false, scapeChar = false;

			// Interpreta las partes de la línea
			foreach (char actual in line)
			{
				// Trata el carácter
				if (isInQuotes)
				{
					if (!scapeChar && (actual == '\\' || actual == '"'))
						scapeChar = true;
					else if (actual == '"')
					{
						if (scapeChar)
						{
							field += actual;
							scapeChar = false;
						}
						else
						{
							isInQuotes = false;
							scapeChar = false;
						}
					}
					else if (actual == FileDefinition.Separator && scapeChar)
					{
						scapeChar = false;
						isInQuotes = false;
						fields.Add(field);
						field = string.Empty;
					}
					else
						field += actual;
				}
				else if (!isInQuotes)
				{
					if (actual == '"')
						isInQuotes = true;
					else if (actual == FileDefinition.Separator)
					{
						// Convierte la cadena
						fields.Add(field);
						// Vacía la cadena intermedia e incrementa el índice del campo
						field = string.Empty;
					}
					else
						field += actual;
				}
				else
					field += actual;
			}
			// Añade el último campo
			fields.Add(field);
			// Devuelve la lista de campos
			return fields;
	}

	/// <summary>
	///		Convierte las cadenas leidas
	/// </summary>
	private List<object?> ConvertFields(List<string> fields)
	{
		List<object?> values = new List<object?>();
		int index = 0;

			// Convierte cada uno de los valores
			foreach (string field in fields)
				if (Columns.Count > index)
					values.Add(ConvertField(Columns[index++], field));
				else
					values.Add(null);
			// Devuelve la lista de valores
			return values;
	}

	/// <summary>
	///		Convierte una cadena en el contenido de una columna
	/// </summary>
	private object? ConvertField(ColumnModel column, string field)
	{
		object? value = null;

			// Convierte una cadena en un objeto
			if (field.Length > 0)
				switch (column.Type)
				{
					case ColumnModel.ColumnType.Unknown:
							value = null;
						break;
					case ColumnModel.ColumnType.DateTime:
							value = field.GetDateTime(FileDefinition.DateFormat);
						break;
					case ColumnModel.ColumnType.Boolean:
							value = field.Equals(FileDefinition.TrueValue, StringComparison.CurrentCultureIgnoreCase);
						break;
					case ColumnModel.ColumnType.Integer:
							value = field.GetInt(0);
						break;
					case ColumnModel.ColumnType.Decimal:
							value = field.ToString().Replace(FileDefinition.DecimalSeparator, '.').GetDouble(0);
						break;
					default:
							value = field.ToString();
						break;
				}
			// Devuelve el valor convertido
			return value;
	}

	/// <summary>
	///		Cierra el archivo
	/// </summary>
	public void Close()
	{
		if (_streamOpenedFromReader && _fileReader != null)
		{
			// Cierra el archivo
			_fileReader.Close();
			// y libera los datos
			_fileReader = null;
		}
	}

	/// <summary>
	///		Obtiene el nombre del campo
	/// </summary>
	public string GetName(int i) => Columns[i].Name;

	/// <summary>
	///		Obtiene el nombre del tipo de datos
	/// </summary>
	public string GetDataTypeName(int i) => GetFieldType(i).Name;

	/// <summary>
	///		Obtiene el tipo de un campo
	/// </summary>
	public Type GetFieldType(int i)
	{
		if (_recordsValues == null)
			return GetColumnType(Columns[i].Type);
		else
			return _recordsValues[i].GetType();
	}

	/// <summary>
	///		Obtiene el tipo de una columna
	/// </summary>
	private Type GetColumnType(ColumnModel.ColumnType type)
	{
		switch (type)
		{
			case ColumnModel.ColumnType.Boolean:
				return typeof(bool);
			case ColumnModel.ColumnType.DateTime:
				return typeof(DateTime);
			case ColumnModel.ColumnType.Integer:
				return typeof(int);
			case ColumnModel.ColumnType.Decimal:
				return typeof(double);
			default:
				return typeof(string);
		}
	}

	/// <summary>
	///		Obtiene el valor de un campo
	/// </summary>
	public object? GetValue(int i) 
	{
		if (i > _recordsValues.Count - 1)
			return null;
		else
			return _recordsValues?[i];
	}

	/// <summary>
	///		Obtiene el esquema
	/// </summary>
	public DataTable GetSchemaTable()
	{
		throw new NotImplementedException();
	}

	/// <summary>
	///		Obtiene una serie de valores
	/// </summary>
	public int GetValues(object[] values)
	{
		throw new NotImplementedException();
	}

	/// <summary>
	///		Obtiene un valor bool de un campo
	/// </summary>
	public bool GetBoolean(int i) => GetDataValue<bool>(i);

	/// <summary>
	///		Obtiene un valor byte de un campo
	/// </summary>
	public byte GetByte(int i) => GetDataValue<byte>(i);

	/// <summary>
	///		Obtiene una serie de bytes
	/// </summary>
	public long GetBytes(int i, long fieldOffset, byte[] buffer, int bufferoffset, int length)
	{
		throw new NotImplementedException();
	}

	/// <summary>
	///		Obtiene un valor char de un campo
	/// </summary>
	public char GetChar(int i) => GetDataValue<char>(i);

	/// <summary>
	///		Obtiene una serie de caracteres
	/// </summary>
	public long GetChars(int i, long fieldoffset, char[] buffer, int bufferoffset, int length)
	{
		throw new NotImplementedException();
	}

	/// <summary>
	///		Obtiene un Guid
	/// </summary>
	public Guid GetGuid(int i) => GetDataValue<Guid>(i);

	/// <summary>
	///		Obtiene un entero de 16
	/// </summary>
	public short GetInt16(int i) => GetDataValue<short>(i);

	/// <summary>
	///		Obtiene un entero de 32
	/// </summary>
	public int GetInt32(int i) => GetDataValue<int>(i);

	/// <summary>
	///		Obtiene un entero largo
	/// </summary>
	public long GetInt64(int i) => GetDataValue<long>(i);

	/// <summary>
	///		Obtiene un valor flotante
	/// </summary>
	public float GetFloat(int i) => GetDataValue<float>(i);

	/// <summary>
	///		Obtiene un valor doble
	/// </summary>
	public double GetDouble(int i) => GetDataValue<double>(i);

	/// <summary>
	///		Obtiene una cadena
	/// </summary>
	public string? GetString(int i)
	{
		object value = GetValue(i);

			if (value is string resultValue)
				return resultValue;
			else
				return value?.ToString();
	}

	/// <summary>
	///		Obtiene un valor decimal
	/// </summary>
	public decimal GetDecimal(int i) => GetDataValue<decimal>(i);

	/// <summary>
	///		Obtiene una fecha
	/// </summary>
	public DateTime GetDateTime(int i) => GetDataValue<DateTime>(i);

	public IDataReader GetData(int i)
	{
		throw new NotImplementedException();
	}

	/// <summary>
	///		Obtiene un campo de un tipo determinado
	/// </summary>
	private TypeData GetDataValue<TypeData>(int i)
	{
		object? value = GetValue(i);

			if (value is TypeData resultValue)
				return resultValue;
			else
				return default;
	}

	/// <summary>
	///		Obtiene el índice de un campo a partir de su nombre
	/// </summary>
	public int GetOrdinal(string name)
	{
		// Obtiene el índice del registro
		if (!string.IsNullOrWhiteSpace(name))
			foreach (ColumnModel column in Columns)
				if (column.Name.Equals(name, StringComparison.CurrentCultureIgnoreCase))
					return Columns.IndexOf(column);
		// Si ha llegado hasta aquí es porque no ha encontrado el campo
		return -1;
	}

	/// <summary>
	///		Indica si el campo es un DbNull
	/// </summary>
	public bool IsDBNull(int index) => index >= _recordsValues?.Count || _recordsValues?[index] is null || _recordsValues[index] is DBNull;

	/// <summary>
	///		Los CSV sólo devuelven un Resultset, de todas formas, DbDataAdapter espera este valor
	/// </summary>
	public bool NextResult() => false;

	/// <summary>
	///		Libera la memoria
	/// </summary>
	protected virtual void Dispose(bool disposing)
	{
		if (!IsDisposed)
		{
			// Libera los datos
			if (disposing)
				Close();
			// Indica que se ha liberado
			IsDisposed = true;
		}
	}

	/// <summary>
	///		Libera la memoria
	/// </summary>
	public void Dispose()
	{
		Dispose(true);
	}

	/// <summary>
	///		Profundidad del recordset
	/// </summary>
	public int Depth => 0; 

	/// <summary>
	///		Indica si está cerrado
	/// </summary>
	public bool IsClosed => _fileReader == null;

	/// <summary>
	///		Registros afectados
	/// </summary>
	public int RecordsAffected => -1;

	/// <summary>
	///		Parámetros de definición del archivo
	/// </summary>
	public FileModel FileDefinition { get; }

	/// <summary>
	///		Columnas
	/// </summary>
	public List<ColumnModel> Columns { get; }

	/// <summary>
	///		Bloque de filas para las que se lanza el evento de grabación
	/// </summary>
	public int NotifyAfter { get; }

	/// <summary>
	///		Indica si se ha liberado el recurso
	/// </summary>
	public bool IsDisposed { get; private set; }

	/// <summary>
	///		Número de campos a partir de las columnas
	/// </summary>
	/// <remarks>
	///		Lo primero que hace un BulkCopy es ver el número de campos que tiene, si no se ha leido la cabecera puede
	///	que aún no tengamos ningún número de columnas, por eso se lee por primera vez
	/// </remarks>
	public int FieldCount 
	{ 
		get 
		{ 
			return Columns.Count; 
		}
	}

	/// <summary>
	///		Indizador por número de campo
	/// </summary>
	public object? this[int i] 
	{ 
		get 
		{ 
			if (_recordsValues is null)
				return null;
			else 
				return _recordsValues[i]; 
		}
	}

	/// <summary>
	///		Indizador por nombre de campo
	/// </summary>
	public object? this[string name]
	{ 
		get 
		{
			if (_recordsValues is null)
				return null;
			else
			{
				int index = GetOrdinal(name);

					if (index >= _recordsValues.Count)
						return null;
					else
						return _recordsValues[GetOrdinal(name)]; 
			}
		}
	}
}
```

Las clases `ColumnModel` y `FileDefinitionModel` contienen la definición de columnas (nombre y tipo) y la definición
del archivo (separadores, formatos de fecha, carácter de punto decimal...). En principio, no tienen importancia para la optimización
y que yo recuerde no han cambiado.

Para comparar los tiempos, ejecuté la consola de *Benchmarks* sobre la clase no optimizada con dos archivos `Sales.csv` con 
1.523.630 registros y `SalesBig.csv` con 6.094.520 registros. Esta será nuestra línea base:

| Method            | FileName     | Mean    | Error    | StdDev   | StdErr   | Min     | Max     | Median  | Ratio | Gen0         | Gen1      | Allocated | Alloc Ratio |
|-------------------|------------- |--------:|---------:|---------:|---------:|--------:|--------:|--------:|------:|-------------:|----------:|----------:|------------:|
| Csv not optimized | Sales.csv    | 1.743 s | 0.0662 s | 0.0102 s | 0.0051 s | 1.728 s | 1.751 s | 1.746 s |  1.00 |  249000.0000 |         - |   3.89 GB |        1.00 |
| Csv not optimized | SalesBig.csv | 6.935 s | 0.1947 s | 0.0506 s | 0.0226 s | 6.862 s | 6.990 s | 6.933 s |  1.00 |  999000.0000 | 3000.0000 |  15.57 GB |        1.00 |

El tiempo parece bastante alto, pero lo más importante en el tipo de optimización que pretendo, es la memoria utilizada. 

Como comentaba, utiliza muchísimo funciones de manejo de cadenas, por eso el uso de memoria es tan alto: las cadenas son inmutables, 
cada creación de cadena va a la memoria, como vemos, prácticamente todo va a la `Gen0` aunque con el segundo archivo, parte de esa 
creación de objetos llega a `Gen1`.

## Proceso de optimización

Hay dos fuentes principales de esa memoria, la primera de ellas es precisamente la interpretación de las líneas
leidas del archivo (el método `ParseLine` del código anterior).

La otra gran fuente de problemas de memoria es la generación de los valores de los campos de salida (el método `ConvertFields`) que
crea una lista de objetos por cada una de las líneas.

Para la primera optimización, cambié el método `ConvertFields` para que no crease esta lista de objetos por línea, definiendo un array global de objetos
que se van a reutilizar en cada línea. 

Parecen optimizaciones acertadas: reducimos el tiempo de lectura casi tres segundos en la lectura del archivo más largo aunque la memoria sigue siendo excesivamente
alta:

| Method            | FileName     | Mean    | Error    | StdDev   | StdErr   | Min     | Max     | Median  | Ratio | Gen0        | Gen1      | Allocated | Alloc Ratio |
|-------------------|------------- |--------:|---------:|---------:|---------:|--------:|--------:|--------:|------:|------------:|----------:|----------:|------------:|
| Csv optimized     | Sales.csv    | 1.044 s | 0.0364 s | 0.0095 s | 0.0042 s | 1.031 s | 1.052 s | 1.047 s |  0.61 | 246000.0000 |         - |   3.84 GB |        0.99 |
| Csv not optimized | Sales.csv    | 1.720 s | 0.0258 s | 0.0040 s | 0.0020 s | 1.715 s | 1.725 s | 1.721 s |  1.00 | 249000.0000 |         - |   3.89 GB |        1.00 |
|                   |              |         |          |          |          |         |         |         |       |             |           |           |             |
| Csv optimized     | SalesBig.csv | 4.345 s | 0.2881 s | 0.0446 s | 0.0223 s | 4.308 s | 4.405 s | 4.333 s |  0.60 | 985000.0000 | 1000.0000 |  15.36 GB |        0.99 |
| Csv not optimized | SalesBig.csv | 7.205 s | 0.1613 s | 0.0419 s | 0.0187 s | 7.134 s | 7.241 s | 7.216 s |  1.00 | 999000.0000 | 3000.0000 |  15.57 GB |        1.00 |

Así que sólo nos queda por utilizar nuestra arma secreta y recurrir a las estructuras `ReadOnlySpan<char>`.

Eso nos va a obligar a modificar el método `ParseLine` del que tampoco estaba excesivamente orgulloso para no utilizar `string`. Este método es el que
crea cada uno de los campos y va añadiendo los caracteres que encuentra a una cadena intermedia:

```csharp
private IEnumerable<string> ParseLine(string line)
{
	string field = string.Empty;
	bool isInQuotes = false, scapeChar = false;

		foreach (char actual in line)
		{
			// Trata el carácter
			if (isInQuotes)
			{
				...
				field += actual;
				...
			}
			else
				field += actual;
		}
		...
}
```

Lo que pretendemos es en lugar de crear la cadena, obtener la longitud de cada campo y utilizar el método `Slice` de `Span` para
quedarnos con la sección de la cadena correspondiente.

El método que calcula la longitud del campo es el siguiente:

```csharp
private int GetLengthField(ReadOnlySpan<char> line, int start)
{
	bool end = false, isInQuotes = false, atScape = false;
	int index = start, length = 0;

		// Calcula la longitud del siguiente campo
		while (index < line.Length && !end)
		{
			char actual = line[index];
			char next = '#';

				// Obtiene el siguiente carácter
				if (index < line.Length - 1)
					next = line[index + 1];
				// Busca el separador
				if (actual == FileDefinition.Separator && !isInQuotes)
					end = true;
				else if (actual == '"')
				{
					// Sea lo que sea añade el carácter a la longitud
					length++;
					// Si es una comilla y lo siguiente es una comilla: se toma como carácter de escape
					if (next != '"')
					{
						// Comprueba si debe cambiar si está entre comillas
						if (atScape)
							atScape = false;
						else
							isInQuotes = !isInQuotes;
					}
					else
						atScape = true;
				}
				else
					length++;
				// Pasa al siguiente carácter
				index++;
		}
		// Devuelve la longitud del campo
		return length;
}
```

Y para obtener el contenido del campo, simplemente utilizamos `line.Slice(start, length)`.

El problema en este caso nos viene con las comillas: la longitud incluye las comillas iniciales y finales que hay quitar del valor, por eso
el método `Normalize` cambia el inicio y la longitud de la zona de memoria:

```csharp
private ReadOnlySpan<char> Normalize(ReadOnlySpan<char> field)
{
	// Si realmente tenemos espacio para tener comillas apreciables
	if (field.Length > 1)
	{
		// Quita las comillas de inicio y fin
		if (field[0] == '"' && field[^1] == '"')
		{
			if (field.Length == 2)
				field = string.Empty;
			else
				field = field[1..^1];
		}
		// Si hay dobles comillas (es decir, al menos hay una comilla dentro) se quita una de ellas
		if (field.IndexOf('"') >= 0)
			field = field.ToString().Replace("\"\"", "\"").AsSpan();
	}
	// Devuelve el campo
	return field;
}
```

En este caso la dificultad es que el valor del campo puede tener comillas intermedias, dado que la estructura `ReadOnlySpan` es de solo lectura,
no me queda más remedio que transformarlo de nuevo en un `String`, reemplazar las dobles comillas por comillas únicas y recuperar el
`Span` de esta cadena, esta conversión nos obliga a volver a crear una cadena, afortunadamente, no es un caso que se dé a menudo (espero).

Por supuesto, el bucle principal también ha cambiado para adecuarlo a estos métodos:

```csharp
private void ConvertFields(ReadOnlySpan<char> line)
{
	int start = 0, length;

		// Convierte los campos
		for (int column = 0; column < _recordsValues.Length; column++)
		{
			// Vacía el valor
			_recordsValues[column] = null;
			// Obtiene el siguiente campo
			length = GetLengthField(line, start);
			// Si hay algo...
			if (length > 0)
			{
				// Obtiene el valor (normaliza la línea para quitar las comillas de inicio a fin)
				_recordsValues[column] = ConvertField(Columns[column], Normalize(line.Slice(start, length)));
				// Asigna el índice de inicio
				start += length + 1;
			}
		}
}
```

Y si ejecutamos nuestro banco de pruebas de nuevo, obtenemos este resultado:

| Method            | FileName     | Mean       | Error     | StdDev   | StdErr   | Min        | Max        | Median     | Ratio | Gen0        | Gen1      | Allocated   | Alloc Ratio |
|-------------------|------------- |-----------:|----------:|---------:|---------:|-----------:|-----------:|-----------:|------:|------------:|----------:|------------:|------------:|
| Csv optimized     | Sales.csv    |   419.4 ms |  10.46 ms |  2.72 ms |  1.21 ms |   416.4 ms |   422.4 ms |   418.6 ms |  0.23 |  42000.0000 |         - |   682.51 MB |        0.17 |
| Csv not optimized | Sales.csv    | 1,845.6 ms |  53.50 ms | 13.89 ms |  6.21 ms | 1,828.0 ms | 1,864.5 ms | 1,845.5 ms |  1.00 | 249000.0000 |         - |  3986.98 MB |        1.00 |
|                   |              |            |           |          |          |            |            |            |       |             |           |             |             |
| Csv optimized     | SalesBig.csv | 1,796.2 ms |  41.83 ms | 10.86 ms |  4.86 ms | 1,783.3 ms | 1,809.3 ms | 1,793.0 ms |  0.25 | 171000.0000 |         - |  2730.02 MB |        0.17 |
| Csv not optimized | SalesBig.csv | 7,282.0 ms | 346.95 ms | 90.10 ms | 40.30 ms | 7,172.9 ms | 7,397.0 ms | 7,275.4 ms |  1.00 | 999000.0000 | 3000.0000 | 15947.87 MB |        1.00 |

Como vemos, la memoria baja en el segundo archivo de 15,57GB a unos 2,66GB, posiblemente derivado de la propia lectura de las líneas de archivo y el tiempo
ha bajado de unos siete segundos a algo menos de dos segundos.

## El código final

Lo último que nos queda es mostrar el código final en [GitHub](https://github.com/jbautistam/LibCsvFiles/blob/main/src/LibCsvFiles/CsvReader.cs).