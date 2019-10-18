[Table of contents](../README.md#contents)

# The Basics

## <a id="logs-runs"></a>Logs and runs

A SARIF log is a JSON file.<sup><a href="#note-1">1</a></sup>
It has a required `version` property that must be `"2.1.0"`.
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
For most tools, the results represent "issues" &mdash; conditions that might detract from the quality of the code.
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

- An identifier for the rule that was violated.
- The severity of the violation.
- A message describing the violation.
- The location of the violation.

There are many other properties used in advanced scenarios, which we'll cover in future tutorials.

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

## <a id="messages"></a>Messages

## <a id="locations"></a>Locations, physical and logical

## <a id="artifacts"></a>Artifacts

## <a id=rule-metadata></a>Rule metadata

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

[Table of contents](../README.md#contents)
