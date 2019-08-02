===============
Troubleshooting
===============

TypeGen.Core is not compatible with your .NET Core project
==========================================================

It's most likely a NuGet issue, which can be resolved by clearing the NuGet cache:

.. code-block:: text

	./nuget.exe locals -clear all
