======================
Command Line Interface
======================

The TypeGen Command Line Interface (CLI) can be used from the Package Manager Console, after installing TypeGen `NuGet package <https://www.nuget.org/packages/TypeGen>`_. The syntax of the command line utility is presented below:

.. code-block:: text

	TypeGen ProjectFolder [-Config-Path "config\path.json"] [Get-Cwd] [-h | -Help] [-v | -Verbose]
	
.. container:: Note

    There is a known issue with using TypeGen from Package Manager Console with ASP.NET Core projects (Package Manager Console doesn't see the TypeGen executable). If this happens to you, you can copy the TypeGen executable from the *packages* folder to your solution directory and use it from PowerShell or CMD.

**Arguments/options**

========================  ======  
ProjectFolder             The path to the project that TypeScript sources will be generated for. Package Manager Console runs from the solution directory by default, so usually the ProjectFolder would be the name of the project (if the project directory has the same name as the project).

-Config-Path              The path to the config file (relative to the project folder). If none specified, *tgconfig.json* file from the ProjectFolder will be used. If *tgconfig.json* is not present, the default configuration will be used.

Get-Cwd                   A utility option. If present, the current working directory of the CLI will be outputted. No file generation will occur when this option is present.

-h or -Help               Shows the help.

-v or -Verbose            The CLI will run in verbose mode. More information will be outputted.
========================  ======

After running the *TypeGen* command, the following actions are performed:

#. The CLI reads the assembly file (.dll or .exe). By default, the assembly file is read from the project folder's *bin/Debug* or *bin* directories (if no assembly path is specified in the config). The name of the assembly must match the name of the .csproj file present in the project folder.

#. Each type in the assembly is checked for the existence of a *ExportTs...* attribute. If an attribute is present for the type, a TypeScript file will be generated.

Configuration file
==================

TypeGen CLI uses a JSON configuration file to read the file generation options. By default, the configuration file is read from the *tgconfig.json* file present in the specified project folder. If *tsconfig.json* does not exist, default options are used. Configuration file path can also be specified by using the *-Config-Path* CLI option. For configuration parameters not present in the configuration file, default values will be used.

The table below shows all available config parameters, with their default values and a description:

============================ =============================== ===================
Parameter                    Default                         Description
============================ =============================== ===================
assemblyPath                 null                            The path to the assembly file with C# types to generate TypeScript sources for. If null or empty, the default strategy for finding an assembly will be used.

fileNameConverters (*)       ["PascalCaseToKebabCase"]       Converter chain used for converting C# type names to TypeScript file names

typeNameConverters (*)       []                              Converter chain used for converting C# type names to TypeScript type names

propertyNameConverters (*)   ["PascalCaseToCamelCase"]       Converter chain used for converting C# property/field names to TypeScript property names

enumValueNameConverters (*)  []                              Converter chain used for converting C# enum value names to TypeScript enum value names

typeScriptFileExtension      "ts"                            File extension for the generated TypeScript files

tabLength                    4                               The number of spaces per tab in the generated TypeScript files

explicitPublicAccessor       false                           Whether to use explicit *public* accessor in the generated TypeScript class files
============================ =============================== ===================

(*) Converter chain is an array of converter class names. The rules for specifying converter chains are as follows:

* Names of converter classes can be specified with or without the *Converter* suffix.

* Names of converter classes can be specified as a class name or a fully qualified class name.

* If only the name of a converter is specified, the converter class will first be searched in the project's assembly and then (if not found) in *TypeGen.Core*.

* To read a converter class from a specific assembly, converter path can be defined in the following format: *assembly/path/assembly.dll:ConverterClass*, where assembly path is relative to the project's folder.

For more information on converters, please refer to the :doc:`Using TypeGen <usingtypegen>` section.

Example
-------

An example of a configuration file (*tgconfig.json*) is presented below:

.. code-block:: json

	{
	    "assemblyPath": "assembly/path/assembly.dll",
	    "fileNameConverters": ["converters/assembly.dll:StripDto", "PascalCaseToKebabCase"],
	    "typeNameConverters": ["converters/assembly.dll:Fqcn.Converters.StripDto"],
	    "propertyNameConverters": [],
	    "enumValueNameConverters": ["UnderscoreCaseToPascalCase"],
	    "typeScriptFileExtension": "ts",
	    "tabLength": 2,
	    "explicitPublicAccessor": true
	}