========
Cookbook
========

Writing your own converter
===============================

Type name and name converter
-------------------

Let's say you want the generated TypeScript files' names to be *kebab-case*, but without the trailing *DTO*, as your C# classes have (let's say you have e.g. a C# class called *ProductDTO*).

One way to do this is to combine the natively available *PascalCaseToKebabCase* converter with some custom converter that strips the *DTO* suffix from the C# class name (if the class name ends with *DTO*). Such converter could be implemented as both *name* converter and *type name* converter, since it doesn't depend on the generated type (it could be reused e.g. for property names conversion).

Here's one implementation of such converter:

.. code-block:: csharp

	public class StripDtoConverter : INameConverter, ITypeNameConverter
	{
	    public string Convert(string name)
	    {
	        return ConvertName(name);
	    }
	    
	    public string Convert(string name, Type type)
	    {
	        return ConvertName(name);
	    }
	    
	    public string ConvertName(string name)
	    {
	        return name.ToUpperInvariant().EndsWith("DTO") ? name.Substring(0, name.Length - 3) : name;
	    }
	}

Now you can create a converter chain that first strips the *DTO* from the C# class name, and later converts the DTO-less name to kebab-case:

* in CLI: specify *["StripDto", "PascalCaseToKebabCase"]* for the *fileNameConverters* parameter in your *tgconfig.json*
* programmatically: create a converter chain: *new TypeNameConverterCollection(new StripDtoConverter(), new PascalCaseToKebabCaseConverter())* and pass it in the generator options

Type name converter
-------------------

Sometimes you may want to make TypeScript file (or type) names dependent on the actual C# type being generated. For example, you may want the enum file names to end with *.enum.ts* or the interface file names to end with *.interface.ts*.

Let's say you're generating all C# classes as TypeScript interfaces (by annotating them with the *ExportTsInterface* attribute). In this case, to achieve the abovementioned conversion, you may write the following type name converter:

.. code-block:: csharp

	public class TypeSuffixConverter: ITypeNameConverter
	{
	    public string Convert(string name, Type type)
	    {
	        if (type.IsClass)
	        {
	            return name + ".interface";
	        }
	        
	        if (type.IsEnum)
	        {
	            return name + ".enum";
	        }
	        
	        return name;
	    }
	}

You could then combine it with the *PascalCaseToKebabCase* converter in the following way:

* in CLI: specify *["PascalCaseToKebabCase", "TypeSuffix"]* for the *fileNameConverters* parameter in your *tgconfig.json*
* programmatically: create a converter chain: *new TypeNameConverterCollection(new PascalCaseToKebabCaseConverter(), new TypeSuffixConverter())* and pass it in the generator options