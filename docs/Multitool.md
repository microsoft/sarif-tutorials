[Table of contents](../README.md#contents)

# Appendix: The SARIF Multitool

The SARIF Multitool is a .NET Core command line tool that performs a variety of operations on SARIF files. It requires the [.NET Core 2.1 Runtime](https://dotnet.microsoft.com/download/dotnet-core/2.1).

With the runtime installed, install the tool as follows:

    dotnet tool install --global Sarif.Multitool

This add a command named `sarif` to your execution path. In a new command window, you can now issue `sarif` commands, such as this one that validates a SARIF file against a set of built-in validation rules:

    sarif validate MyFile.sarif --output MyFile-validation.sarif

[Table of contents](../README.md#contents)