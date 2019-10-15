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

## <a id="results"></a>Results

The primary purpose of a run is to hold a set of "results".
A result is an observation about the code.
For most tools, the results represent "issues" &mdash; conditions that might detract from the quality of the code.
But some results might be purely informational.

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

## <a id="messages"></a>Messages

## <a id="locations"></a>Locations, physical and logical

## <a id="artifacts"></a>Artifacts

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
