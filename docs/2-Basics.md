[Table of contents](../README.md#contents)

# The Basics

## <a id="log-file-and-om"></a>The log file and the object model

A SARIF log is a JSON file.<sup><a href="#note-1">1</a></sup>
The SARIF spec defines an _object model_ to describe the contents of this file,
and the top-level object -- the object that represents the log file as a whole --
is the `sarifLog` object.

To work with the contents of a log file in your program,
you need a set of classes that correspond to the elements of the SARIF object model.
The SARIF spec doesn't standardize the _bindings_ between its object model and any particular programming language.
Today, there are bindings for .NET (available in the [SARIF SDK](https://www.nuget.org/packages/Sarif.Sdk/) NuGet package)
and Python (available in the [`sarif-om`](https://pypi.org/project/sarif-om/) Python module).
Each language binding conforms to normal language conventions.
So, for example, the `sarifLog` SARIF object is represented by the C# class `SarifLog` in the SARIF SDK,
and by the Python class `sarif_log` in the `sarif-om` Python module.

When we discuss the SARIF format, we usually speak in terms of its object model;
for example, we might talk about the required properties of the `result` object,
or about various places where `message` objects are used.
So if we tell you that a particular property in the JSON file is a `message` object,
you know what its sub-structure looks like without having to say anything more.

## <a id="logs-runs"></a>Logs and runs

The `sarifLog` object has a required `version` property that must be `"2.1.0"`.
`version` should come first (even though JSON is insensitive to property order)
so SARIF consumers such as viewers can "sniff" the version.

The optional `$schema` property contains the URI of a copy of the SARIF schema.
This allows development environments like VS Code to provide schema validation
(for example, squiggles under misspelled property names) and Intellisense.

```json
{
  "version": "2.1.0",
  "$schema": "https://schemastore.azurewebsites.net/schemas/json/sarif-2.1.0-rtm.4.json",
  ...
}
```

A log file contains an array of one or more runs.<sup><a href="#note-2">2</a></sup>
Each run represents a single invocation of a single analysis tool,
and the run has to describe the tool that produced it.

```json
{
  "version": "2.1.0",
  "$schema": "https://schemastore.azurewebsites.net/schemas/json/sarif-2.1.0-rtm.4.json",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "CodeScanner"
        }
      },
      ...
    }
  ]
}
```

Usually there is only one run. SARIF allows multiple runs for convenience,
so that, for example, you can send the runs over a network in a single request.<sup><a href="#note-3">3</a></sup>

The sub-property `tool.driver` describes the tool's "primary executable".
Usually, that's enough, but some tools support plugins &mdash; for example,
code libraries that define additional analysis rules.
SARIF defines the optional `tool.extensions` property to represent plugins.<sup><a href="#note-4">4</a></sup>

If the tool didn't detect any problems, the log file might look like this:

```json
{
  "version": "2.1.0",
  "$schema": "https://schemastore.azurewebsites.net/schemas/json/sarif-2.1.0-rtm.4.json",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "CodeScanner"
        }
      },
      "results": []
    }
  ]
}
```

## <a id="results"></a>Results

The primary purpose of a run is to hold a set of "results".
A result is an observation about the code.
For most tools, the results represent _issues_ &mdash; conditions that might detract from the quality of the code.
But some results might be purely informational.

```json
{
  "version": "2.1.0",
  "$schema": "https://schemastore.azurewebsites.net/schemas/json/sarif-2.1.0-rtm.4.json",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "CodeScanner"
        }
      },
      "results": [
        {
          "ruleId": "no-unused-vars",
          "level": "error",
          "message": {
            "text": "'x' is assigned a value but never used."
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "file:///C:/dev/sarif/sarif-tutorials/samples/Introduction/simple-example.js",
                  "index": 0
                },
                "region": {
                  "startLine": 1,
                  "startColumn": 5
                }
              }
            }
          ]
        }
      ]
    }
  ]
}
```

The most commonly used properties of a result are:

- A message describing the violation.
- An identifier for the rule that was violated.
- The severity of the violation.
- The location of the violation.

There are many other properties used in advanced scenarios, which we'll cover in future tutorials.

When you open a SARIF file in a typical SARIF viewer, the viewer will display the list of results.
For example, the
[Microsoft SARIF Viewer VSIX for Visual Studio](https://marketplace.visualstudio.com/items?itemName=WDGIS.MicrosoftSarifViewer)
displays the results in Visual Studio's Error List window.
If [physical location information](#phys-log-loc) is available,
then when the user selects one of the results (in Visual Studio, by double-clicking it),
the viewer will navigate to the result's location
by opening the file specified by `physicalLocation.artifactLocation.uri` (`simple-example.js` in the example above).
The viewer will typically scroll the portion of the file specified by `physicalLocation.region` (line 1 in the example)
into view, and highlight it.

## <a id="messages"></a>Messages

The only required property of a `result` object is the `message` property.

SARIF messages are more than just strings: they are represented by a separate object, the `message` object.
In the simplest case, which we'll show here, a `message` object contains a simple text string, like this:

```json
{
  "message": {
    "text": "'x' is assigned a value but never used."
  }
}
```

## <a id=rule-id></a>Rule identifier

Most tools provide a "code" for each rule, for example `CA1304`, the Roslyn analyzer code
for the rule "Specify CultureInfo".
The SARIF property `result.ruleId`<sup><a href="#note-5">5</a></sup> holds this code.

```json
{
  "ruleId": "CA1304"
}
```

The spec explains why `result.ruleId` should be a "stable, opaque" identifier.
It shouldn't change over time so that build scripts that disable a particular rule never break.
It should be opaque (as opposed to being a human-readable string) for ease of web lookup
and to avoid language difficulties.

Not all tools provide such an identifier.
For example, ESLint uses human-readable rule identifiers such as `"no-unused-vars"`.
These tools should do the best they can to populate `result.ruleId`.

## <a id=level></a>Level

The `result.level` property says how serious the result is. It usually has one of three values:

- "error": A serious problem
- "warning": A problem
- "note": A minor problem or an opportunity to improve the code

```json
{
  "ruleId": "CA1304",
  "level": "warning"
}
```

There are advanced scenarios where `result.level` has the value `"None"`, but we won't discuss them here.<sup><a href="#note-6">6</a></sup>

In the simplest case, `result.level` defaults to `"warning"`.
But the complete algorithm for determining the default is complicated
because it takes into account certain advanced scenarios<sup><a href="#note-7">7</a></sup>.
For that reason, you are IMO better off just specifying it explicitly in each result,
as in the example above.

## <a id="locations"></a>Locations

### <a id="loc-array"></a>The `locations` array

A SARIF `result` almost always contains a `locations` property whose value is an array of `location` objects,
and that array almost always contains exactly one element:

```json
{
  "ruleId": "CA1304",
  "level": "warning",
  "locations": [
    {
      ...
    }
  ]
}
```

`result.locations` is optional because a location doesn't always sense.
For example, if a tool tells you that your C# program _doesn't have_ a `Main` entry point, what location
should it mention?

`result.locations` is an array because sometimes you have to make changes in more than one place to fix a problem.
Suppose a style checker tells you that your C# class name doesn't start with a capital letter,
and suppose your class is made up of a set of `partial` classes.
You can't change the name in just one place: your code won't compile.
You have to change every occurrence, and that's why the `result` points you at all the occurrences.
(Of course IDEs can help you with this.)

### <a id="phys-log-loc"></a>Physical and logical locations

SARIF supports two kinds of locations: physical and logical.
A _physical location_ describes a location with respect to some programming artifact,
for example, a range of lines in a source file or a byte range in an executable file.
A _logical location_ describes a location by name, without reference to a programming artifact,
for example the name of a method within a class within a namespace.

SARIF supports logical locations for two reasons:

1. Most important, binary analysis tools don't always have physical location information available
(although they might, for example, if they have access to a symbols file).
These tools have no choice but to report (for example) a method name;
they can't tell you what file the method was defined in.

2. Even if physical location is available, it can be helpful for a tool to tell you that the problem
on line 42 is inside function `f()`.

The most common case is for a tool to report a physical location (rather than a logical location),
and to specify the location by line and column number rather (rather than as a byte range):

```json
{
  "locations": [
    {
      "physicalLocation": {
        "artifactLocation": {
          "uri": "io/kb.c"
        },
        "region": {
          "startLine": 42,
          "startColumn": 9
        }
      }
    }
  ]
}
```

A `physicalLocation` object almost always contains an `artifactLocation` property,<sup><a href="#note-8">8</a></sup>
and it can also contain a `region` property.

A simple logical location looks like this:

```json
{
  "locations": [
    {
      "logicalLocations": [
        {
          "fullyQualifiedName": "NamespaceN.ClassC.MethodM"
        }
      ]
    }
  ]
}
```

Note that `location.logicalLocations` is an array.
As usual, this is to support an advanced scenario.<sup><a href="#note-9">9</a></sup>

## <a id="artifacts"></a>Artifacts

An _artifact_ is anything you create in the course of programming, such as a source file or a web page.
In SARIF, every artifact must be URL-addressable.
That means, for example, that if you want to write a static database analyzer that produces SARIF,
then you need a way to express the locations of the things that you analyze -- tables, rows, indices, and so on --
as URLs.

### <a id="defining-artifacts"></a> Defining artifacts

As we said earlier, almost every result specifies a location, and those locations are often physical locations
which in turn contain `artifactLocation` objects:

```json
{
  "locations": [
    {
      "physicalLocation": {
        "artifactLocation": {
          "uri": "io/kb.c"
        },
        ...
      }
    }
  ]
}
```

At this point, all you know about the artifact is its location.
But SARIF lets you provide more information about each artifact by using the `run.artifacts` property,
whose value is an array of `artifact` objects:

```json
{
  "runs": [
    {
      "artifacts": [
        {
          "location": {
            "uri": "io/kb.c"
          },
          "length": 3444,
          "sourceLanguage": "c",
          "hashes": {
            "sha-256": "b13ce2678a8807ba0765ab94a0ecd394f869bc81"
          }
        }
      ]
    }
  ]
}
```

Note that:

- `length` is measured in bytes.
- The SARIF spec suggests values for the `sourceLanguage` property for many programming languages.<sup><a href="#note-10">10</a></sup>
- `hashes` can contain any number of hashes calculated by different algorithms.
The spec recommends<sup><a href="#note-11">11</a></sup> using the algorithms and algorithm names contained in the
[IANA registry of hash function names](https://www.iana.org/assignments/hash-function-text-names/hash-function-text-names.xhtml),
and it particularly recommends that the `hashes` object contain a property `"sha-256"`.

There are more properties to explore.
In particular, you can embed the entire contents of each artifact in the SARIF file,
which allows people to view the results in context even if they're not enlisted in the code base that was analyzed:

```json
{
  "runs": [
    {
      "artifacts": [
        {
          "location": {
            "uri": "io/kb.c"
          },
          "contents": {
            "text": "#include <stdio>\n#include<stack>..."
          }
        }
      ]
    }
  ]
}
```

Of course that requires tooling that can extract the source file contents from the log file and display them.

### <a id="linking-artifacts"></a> Linking results to artifacts

## <a id="rule-metadata"></a>Rule metadata

## Notes

<a id="note-1"></a>1. In future, SARIF might support other serializations of its underlying object model.
See [§3.1](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012376).

<a id="note-2"></a>2. In rare cases, `runs` can be empty or even `null`.
See [§3.13.4: runs property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012438)
for more information.

<a id="note-3"></a>3. There's also an advanced scenario where multiple runs can share data stored at the log level,
reducing the total size of the payload. See [§3.13.5 inlineExternalProperties property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012439)
for an example.

<a id="note-4"></a>4. See [§3.18.3, extensions property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012488).

<a id="note-5"></a>5. We use the notation _objectName_._propertyName_, so `result.ruleId` denotes the `ruleId` property
of the SARIF `result` object.

<a id="note-6"></a>6. For more information, see
[§3.27.10, level property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012604)
and
[§3.27.9, kind property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012603).

<a id="note-7"></a>7. Again, see
[§3.27.10, level property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012604)
and
[§3.27.9, kind property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012603).

<a id="note-8"></a>8. `physicalLocation.artifactLocation` isn't required because SARIF also allows you to
specify a location by its address (see
[§3.29.6, address property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012640) and
[§3.32, address object](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012661)).
This supports binary analysis tools.
It also supports tools that examine memory contents.
This isn't something a static analysis tool would do,
but as I mentioned earlier, SARIF actually has some level of support for dynamic analysis tools,
although the spec never makes that claim.

<a id="note-9"></a>9. For more information, see
[§3.29.6, logicalLocations property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012630).

<a id="note-10"></a>10. See
[Appendix J. (Informative) Sample sourceLanguage values](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012903).

<a id="note-10"></a>10. See
[§3.24.11, hashes property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012580)

[Table of contents](../README.md#contents)
