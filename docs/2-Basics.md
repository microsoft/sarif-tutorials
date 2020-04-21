[Table of contents](../README.md#contents)

# The Basics

## <a id="log-file-and-om"></a>The log file and the object model

A SARIF log is a JSON file.<sup><a href="#note-1">1</a></sup>
The SARIF spec defines an <a href="Glossary.md#object-model">_object model_</a>
to describe the contents of this file,
and the top-level object &mdash; the object that represents the log file as a whole &mdash;
is the `sarifLog` object.

To work with the contents of a log file in your program,
you need a set of classes that correspond to the elements of the SARIF object model.
The SARIF spec doesn't standardize the <a href="Glossary.md#binding">_binding_</a> between its object model
and any particular programming language.
Today, there are bindings for .NET (in the [SARIF SDK](https://www.nuget.org/packages/Sarif.Sdk/) NuGet package)
and Python (in the [`sarif-om`](https://pypi.org/project/sarif-om/) Python module).
Each language binding conforms to normal language conventions.
For example, the `sarifLog` SARIF object is represented by the C# class `SarifLog` in the SARIF SDK,
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

## <a id="tools"></a> Tools: driver and extensions

The `tool` property is required. It describes the analysis tool that produced the run.
The sub-property `tool.driver` is also required. It describes the tool's
<a href="Glossary.md#driver">_driver_</a>, which is the
<a href="Glossary.md#tool-component">_tool component_</a> that contains the tool's
<a href="Glossary.md#primary-executable">_primary executable_</a>.

Some tools support additional components called
<a href="Glossary.md#extension">_extensions_</a> (a.k.a. "plugins"),
for example, code libraries that define additional analysis rules.
SARIF defines the optional `tool.extensions` property
to represent extensions.<sup><a href="#note-4">4</a></sup>

Finally, `tool.driver.name` is required. Everything else under `tool` and `tool.driver` is optional.

## <a id="property-bags"></a>Property bags

Before we go any further, let's address an issue that almost every tool vendor cares about:
What do I do if my tool produces information that the SARIF specification doesn't mention?

The answer is that every object in the SARIF object model &mdash; from logs to runs to results to locations
to messages, without exception &mdash; defines a property named `properties`.
The spec calls a property named `properties` a <a href="Glossary.md#property-bag">_property bag_</a>.

A property bag is a set of name/value pairs &mdash; a JSON object, in SARIF's JSON serialization &mdash;
with any name and any value.
The values can be integers, Booleans, arrays, nested objects &mdash; anything at all.
If you can't find an element of your tool's data in the SARIF specification, the property bag is your friend.

For example, the [Fortify](https://www.microfocus.com/en-us/products/application-security-testing/overview)
tool from [MicroFocus](https://www.microfocus.com/) assesses the "confidence" of each result &mdash;
the likelihood that the result is a "true positive."
It looks like this:

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
          ...
          "properties": {
            "Confidence": 5.0
          }
        }
      ]
    }
  ]
}
```

Having said all that, it's important that you do your best to use the properties that SARIF defines
(we call them <a href="Glossary.md#first-class-property">_first class properties_</a>)
rather than using the property bag.
Generic SARIF tooling &mdash; tooling that is not aware of the details of any particular tool &mdash;
will at best be able to display property bag properties.
It won't be able to extract any meaning from them.

There's a balancing act here, because it's also problematic to populate SARIF's first class properties
with information that doesn't match their semantics.
Each tool vendor needs to make the call on a property by property basis.

## <a id="results"></a>Results

The primary purpose of a run is to hold a set of <a href="Glossary.md#result">_results_</a>.
A result is an observation about the code.
For most tools, the results represent <a href="Glossary.md#issue">_issues_</a> &mdash;
conditions that might detract from the quality of the code &mdash;
but some results might be purely informational.

In this example, the tool produced one result:

```json
{
  "version": "2.1.0",
  "$schema": "https://schemastore.azurewebsites.net/schemas/json/sarif-2.1.0-rtm.4.json",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "ESLint"
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

### <a id="message"></a>Message

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

We'll say much [more about the features and capabilities of messages](3-Beyond-basics.md#more-about-messages) later.
Just as important as these technical issues is the quality of the message text.
[Appendix A](Authoring-rule-metadata-and-result-messages.md) provides guidance on authoring
informative and actionable result messages.

### <a id=rule-id></a>Rule identifier

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

### <a id=level></a>Level

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

### <a id="locations"></a>Locations

#### <a id="loc-array"></a>The `locations` array

We call a place in the code where a tool detects a result a
<a href="Glossary.md#result-location">_result location_</a>.
SARIF represents result locations with the optional property `result.locations`,
an array of `location` objects which almost always contains exactly one element:

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

Don't use `result.locations` to specify the locations of multiple problems, even problems of the same kind,
if they can be fixed independently.
You might choose to fix some occurrences of the problem and not others, for example,
if you know that some of the occurrences are in code that is slated for removal,
or are false positives.
Only put more than one element in `result.locations` if you _have_ to fix all the locations at once.

#### <a id="phys-log-loc"></a>Physical and logical locations

SARIF supports two kinds of locations: physical and logical.
A <a href="Glossary.md#physical-location">_physical location_</a>
describes a location with respect to some programming artifact,
for example, a range of lines in a source file or a byte range in an executable file.
A <a href="Glossary.md#logical-location">_logical location_</a>
describes a location by name, without reference to a programming artifact,
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

An <a href="Glossary.md#artifact">_artifact_</a> is anything you create in the course of programming,
such as a source file, an object file, or a web page.
In SARIF, every artifact must be URL-addressable.
That means, for example, that if you want to write a static database analyzer that produces SARIF,
then you need a way to express the locations of the things that you analyze &mdash; tables, rows, indices, and so on &mdash;
as URLs.

The SARIF spec uses the term "artifact" in preference to "file" to emphasize that SARIF doesn't just support tools
that analyze files.
Having said that, in these tutorials we'll occasionally lapse and use the word "file" because it's just too
awkward to say things like "open the artifact" when "open the file" is so much more natural.

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

Now that we have a result that occurs in an artifact,
and we have more information about the artifact,
how can we link the result to the artifact?
For example, if a user is examining a result in a SARIF viewer,
how can the viewer present information about the file where the result occured?

It would be natural to think that you could just find the element of `run.artifacts` whose `location.uri`
matches the result's `physicalLocation.artifactLocation.uri`.
In fact, early drafts of the SARIF spec did just that: `run.artifacts` was a JSON object whose property names
were the URIs.

The problem is that in obscure cases, two distinct artifacts can have the same URI.
So SARIF establishes the link from a result to an artifact by using the `artifactLocation.index` property, like this:

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
      ],
      "results": [
        {
          "message": {
            "text": "Variable 'x' is used before being initialized."
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "io/kb.c",
                  "index": 0
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

The `"index": 0` says "To find more information about the artifact at this location, look at the `artifact` object
at index 0 in the array `run.artifacts`."

When `artifactLocation.index` is present, `artifactLocation.uri` is redundant,
because you can find it in the linked `artifact` object.
A tool can choose to omit `uri` to make the log file smaller,
or to include it to make the file more understandable to a human reader.<sup><a href="#note-12">12</a></sup>

There are many places in SARIF where a property named `index` (or sometimes a more specific name, like `ruleIndex`)
establishes a link from a SARIF object to another object that resides in an array.
For each such property, the spec explains which array to look in.

## <a id="rule-metadata"></a>Rule metadata

A SARIF log file can contain information about the analysis rules defined by the static analysis tool.
The spec refers to this information as <a href="Glossary.md#rule-metadata">_rule metadata_</a>.
Rule metadata can include a complete description of the rule,
its default severity level,
one or more message strings (possibly including substitution sequences like `{0}`) to include in a result),
and a URI where you can find more information about the rule.

If rule metadata is present, then when a user selects a result in a SARIF file,
a SARIF viewer can display the metadata for the rule that was violated.
Here is a screen shot that shows the
[Microsoft SARIF Viewer VSIX for Visual Studio](https://marketplace.visualstudio.com/items?itemName=WDGIS.MicrosoftSarifViewer)
displaying the SARIF file shown in the [simple example](1-Introduction.md#simple-example-file) from the introduction.
The user has selected the result in the Error List window at the bottom.
On the right, the user has selected the Info tab in the SARIF Explorer,
and viewer has displayed the help URI from the metadata for the `no-unused-vars` rule.

![A SARIF viewer displays rule metadata for a result](../images/rule-metadata-for-a-result.png)

[Appendix A](Authoring-rule-metadata-and-result-messages.md) provides guidance on authoring
rule metadata that provides the most useful information to the developer
and also works well in automated systems.

Note the presence of a `ruleIndex` property in the `result` object in the example.
In the same way that `result.locations[0].physicalLocation.artifactLocation.index` provides a link between
a result and the artifact it was found in,
`result.ruleIndex` provides the link between a result and the metadata for the rule that was violated.
The link can't be made through `result.ruleId` because some tools use the same identifier for distinct rules,
each with their own metadata.

In the same way that `artifactLocation.uri` is unnecessary if `artifactLocation.index` is present,
so `result.ruleId` is unnecessary if `result.ruleIndex` is present.
The same considerations apply when deciding whether to include `result.ruleId`:
omitting it makes the log file smaller, while including it makes the log file more understandable to a human user.

Rule metadata is optional.
An analysis tool can choose not to include it at all,
to include metadata for only those rules that are relevant to the results,
or to include metadata for all rules known to the tool.<sup><a href="#note-13">13</a></sup>

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

<a id="note-11"></a>11. See
[§3.24.11, hashes property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012580)

<a id="note-12"></a>12. Rather than requiring every analysis tool to implement logic for excluding redundant properties
to reduce file size, or including them to improve readability, such "file transformation" operations can be
implemented by a <a href="Glossary.md#post-processor">_post-processor_</a>.
The `Sarif.Multitool` NuGet package include a command line tool that (among other things) can post-process
SARIF files, although at the time of this writing it doesn't implement the exact operation I've described here.

<a id="note-13"></a>13.
[Appendix E. (Informative) Locating rule and notification metadata](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012891)
discusses how to decide whether to include rule metadata in a SARIF log file.

[Table of contents](../README.md#contents)
