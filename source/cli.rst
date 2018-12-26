======================
Command Line Interface
======================

The TypeGen Command Line Interface (CLI) can be used either from the Package Manager Console, after installing TypeGen `NuGet package <https://www.nuget.org/packages/TypeGen>`_, or as a `.NET CLI tool <https://www.nuget.org/packages/TypeGen.DotNetCli>`_.

.. container:: Note

    **Note: This issue has been resolved in Visual Studio 2017 RC**
	
    There is a known issue with using TypeGen from Package Manager Console with ASP.NET Core projects, **under Visual Studio 2015**: Package Manager Console doesn't see the TypeGen executable. If this happens to you, you can copy the TypeGen executable from the *packages* folder to your solution directory and use it from PowerShell or CMD.

After running *[dotnet ]typegen* command, the following set of actions is performed for each of the specified project folders:

#. The CLI reads the assembly file (.dll or .exe). By default (if no assembly path is specified in the config), the assembly file is searched recursively in the project folder's *bin* directory. The name of the assembly must match the name of the .csproj or .xproj file present in the project folder.

#. Each type in the assembly is checked for the existence of a *ExportTs...* attribute. If an attribute is present for the type, a TypeScript file will be generated.
	
New syntax (>= 2.0.0)
=====================

.. code-block:: text

	[dotnet-]typegen [options] [command]
	
	Options:
	-h|--help               Show help information
	-v|--verbose            Show verbose output
	-p|--project-folder     Set project folder path(s)
	-c|--config-path        Set config path(s) to use
	
	Commands:
	generate     Generate TypeScript files
	getcwd       Get current working directory

Old syntax (1.x.x)
==================
	
.. code-block:: text

	TypeGen ProjectFolder1[|ProjectFolder2|(...)] [-Config-Path path1[|path2|(...)]] [Get-Cwd] [-h | -Help] [-v | -Verbose]

**Arguments/options**

========================  ======  
ProjectFolderN            The path to project(s) that TypeScript sources will be generated for. Package Manager Console runs from the solution directory by default, so usually ProjectFolderN would be the name of the project.

-Config-Path              Paths to config files for the specified projects (the order of config paths must match the order of projects). If a project doesn't have its config specified, *tgconfig.json* file from ProjectFolderN will be used. If *tgconfig.json* is not present, the default configuration will be used.

Get-Cwd                   A utility option. If present, the current working directory of the CLI will be outputted. No file generation will occur when this option is present.

-h or -Help               Shows the help.

-v or -Verbose            The CLI will run in verbose mode. More information will be outputted.
========================  ======

Configuration file
==================

TypeGen CLI uses a JSON configuration file to read the file generation options. By default, the configuration file is read from the *tgconfig.json* file present in the specified project folder. If *tgconfig.json* does not exist, default options are used. Configuration file path can also be specified by using the *--config-path* CLI option. For configuration parameters not present in the configuration file, their default values are used upon file generation.

The table below shows all available config parameters, with their default values and a description:

============================ =============================== ===================
Parameter                    Default                         Description
============================ =============================== ===================
assemblies                   []                              An array of paths to assembly files to generate TypeScript sources from. If null or empty, the default strategy for finding an assembly will be used. **Note:** the order of assemblies also determines the order of reading any custom converters.

**(DEPR.)** assemblyPath     null                            The path to the assembly file with C# types to generate TypeScript sources for. If null or empty, the default strategy for finding an assembly will be used.

fileNameConverters (*)       ["PascalCaseToKebabCase"]       Converter chain used for converting C# type names to TypeScript file names

typeNameConverters (*)       []                              Converter chain used for converting C# type names to TypeScript type names

propertyNameConverters (*)   ["PascalCaseToCamelCase"]       Converter chain used for converting C# property/field names to TypeScript property names

enumValueNameConverters (*)  []                              Converter chain used for converting C# enum value names to TypeScript enum value names

externalAssemblyPaths        []                              An array of paths to external assemblies. These paths are searched (recursively) for any assembly references that cannot be automatically resolved. NuGet package folders (global + machine-wide and project fallback) are searched by default.

typeScriptFileExtension      "ts"                            File extension for the generated TypeScript files

tabLength                    4                               The number of spaces per tab in the generated TypeScript files

explicitPublicAccessor       false                           Whether to use explicit *public* accessor in the generated TypeScript class files

singleQuotes                 false                           Whether to use single quotes for string literals in the generated TypeScript files

addFilesToProject            false                           **Only for .NET Framework apps (not .NET Core)**. Whether to add the generated TypeScript files to the project file (\*.csproj)

outputPath                   ""                              Output path for generated files, relative to the project folder.

createIndexFile              false                           Whether to generate an index (barrel) file in the root TypeScript output directory. Index exports everything from all generated TypeScript files.

strictNullChecks             false                           Whether to enable TypeScript2 strict null checking mode functionality.

csNullableTranslation        ""                              **Only for strict null checking**. Determines how C# nullable property types will be translated to TypeScript by default. Possible values: "null", "undefined", "null|undefined" or "".

defaultValuesForTypes        null                            Object containing a map of default values for the specified TypeScript types (example below)

customTypeMappings           null                            Object containing a map of custom [C# to TypeScript] type mappings (example below)
============================ =============================== ===================

(*) Converter chain is an array of converter class names. The rules for specifying converter chains are as follows:

* Names of converter classes can be specified with or without the *Converter* suffix.

* Names of converter classes can be specified as a class name or a fully qualified class name.

* If only the name of a converter is specified, the converter class will first be searched in the project's assembly and then (if not found) in *TypeGen.Core*.

* To read a converter class from a specific assembly, converter path can be defined in the following format: *assembly/path/assembly.dll:ConverterClass*, where assembly path is relative to the project's folder.

For more information on converters, please refer to the :doc:`Coverters <converters>` section.

Example
-------

An example of a configuration file (*tgconfig.json*) is presented below:

.. code-block:: json

	{
	    "assemblies": ["my/app/MyApp.Web.dll", "my/app/MyApp.Models.dll"],
	    "fileNameConverters": ["converters/MyApp.Converters.dll:StripDto", "PascalCaseToKebabCase"],
	    "typeNameConverters": ["converters/MyApp.Converters.dll:Fqcn.Converters.StripDto"],
	    "propertyNameConverters": [],
	    "enumValueNameConverters": ["UnderscoreCaseToPascalCase"],
	    "typeScriptFileExtension": "ts",
	    "tabLength": 2,
	    "explicitPublicAccessor": true,
		"defaultValuesForTypes": {
	        "number": "-1",
	        "Date | null": "null",
	        "string": "\"\""
	    },
	    "customTypeMappings": {
	        "System.DateTime": "string",
	        "Some.Custom.Type": "number"
	    }
	}