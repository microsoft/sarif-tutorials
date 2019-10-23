[Table of contents](../README.md#contents)

# Beyond the Basics

## <a id="related-locations"></a>Related locations

Sometimes there are places in the code other than the result location that can can help you understand a problem.
We call these places <a href="5.2-Glossary.md#related-location">_related locations_</a>.
SARIF represents related locations with the optional `result.relatedLocations` property,
an array of `location` objects.

For example, suppose you analyze this Python file:

```python
# 3-Beyond-basics/bad-eval.py

expr = input("Expression> ")
print(eval(expr))
```

The tool might detect the use of `eval` on a "tainted" variable
(one that entered the system through user input and wasn't subsequently "sanitized"),
and might produce a `result` like this (see [../samples/3-Beyond-basics/bad-eval.sarif](../samples/3-Beyond-basics/bad-eval.sarif)):

```json
{
  "ruleId": "PY2335",
  "message": "Use of tainted variable 'expr' in the insecure function 'eval'.",
  "locations": [
    {
      "physicalLocation": {
        "artifactLocation": {
          "uri": "3-Beyond-basics/bad-eval.py"
        },
        "region": {
          "startLine": 4
        }
      }
    }
  ]
}
```

In a large code base, a user might not immediately see where the variable `expr` came from
or why it is considered tainted.
`result.relatedLocations` can help
(see [../samples/3-Beyond-basics/bad-eval-related-locations.sarif](../samples/3-Beyond-basics/bad-eval-related-locations.sarif)):

```json
{
  "ruleId": "PY2335",
  "message": "Use of tainted variable 'expr' in the insecure function 'eval'.",
  "locations": [
    {
      "physicalLocation": {
        "artifactLocation": {
          "uri": "3-Beyond-basics/bad-eval.py"
        },
        "region": {
          "startLine": 4
        }
      }
    }
  ],
  "relatedLocations": [
    {
      "message": {
        "text": "The tainted data entered the system here."
      },
      "physicalLocation": {
        "artifactLocation": {
          "uri": "3-Beyond-basics/bad-eval.py"
        },
        "region": {
          "startLine": 3
        }
      }
    }
  ]
}
```

The `message` property of the related location helps the user understand the problem.

## <a id="more-about-messages"></a>More about messages

We have seen that in its simplest usage, a `message` object has a `text` property and that's the end of it.
But `message` objects can do much more.

### <a id="msg-markdown"></a> Markdown messages

A `message` object can optionally have a `markdown` property containing a string formatted with GitHub-Flavored
Markdown (GFM).
Not every SARIF viewer will know how to render GFM, so while this is legal:

```json
"message": {
  "text": "This is great!"
}
```

... and this is legal:

```json
"message": {
  "text": "This is great!",
  "markdown": "This is _great_!"
}
```

... this is _not_:

```json
"message": {
  "markdown": "This is _illegal_ because there's no `text` property!"
}
```

### <a id="msg-metadata"></a>Messages from metadata

Some result messages are long, because a good message not only explains what was wrong:
it also (as appropriate) explains why the flagged construct is considered questionable,
provides guidance for remedying the problem,
and explains when it's ok to ignore the result.

To avoid repeating the lengthy message in every result, a `message` object can specify an
identifier for the message text
(see [../samples/3-Beyond-basics/message-from-metadata.sarif](../samples/3-Beyond-basics/message-from-metadata.sarif)):

```json
{
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "CodeScanner",
          "rules": [
            {
              "id": "CS0001",
              "messageStrings": {
                "default": {
                  "text": "This is the message text. It might be very long."
                }
              }
            }
          ]
        }
      },
      "results": [
        {
          "ruleId": "CS0001",
          "ruleIndex": 0,
          "message": {
            "id": "default"
          }
        }
      ]
    }
  ]
}
```

The `message.id` property tells us to look in the rule metadata for a string named `default`.
The `result.ruleIndex` property tells us _which_ rule's metadata to look at (the rule at index 0
in `tool.driver.rules`).
The desired message is the property of `messageStrings` whose name matches `id`;
that is, the property named `default`.<sup><a href="#note-1">1</a></sup>

This is another of the file-size-vs.-readability tradeoffs that SARIF offers.
If rule metadata is available, a tool (or a <a href="5.2-Glossary.md#post-processor">_post-processor_</a>)
can choose to inline the messages or to refer to them in the metadata.

### <a id="msg-args"></a>Messages with arguments

Some messages vary from result to result because (for example) they mention specific variables:

```text
Variable 'x' was used without being initialized.
```

Messages in metadata can include C#-like placeholders (`{0}`).
If an analysis tool creates `message` objects that refer to message strings in metadata,
it must provide values for the placeholders by populating the `arguments` property
(see [../samples/3-Beyond-basics/message-with-arguments.sarif](../samples/3-Beyond-basics/message-with-arguments.sarif)):

```json
{
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "CodeScanner",
          "rules": [
            {
              "id": "CS0001",
              "messageStrings": {
                "default": {
                  "text": "Variable '{0}' was used without being initialized."
                }
              }
            }
          ]
        }
      },
      "results": [
        {
          "ruleId": "CS0001",
          "ruleIndex": 0,
          "message": {
            "id": "default",
            "arguments": [
              "x"
            ]
          }
        }
      ]
    }
  ]
}
```

The elements of the `arguments` array are strings!
Arguments of other types must be converted to strings before being added to `arguments`.
It's up to the tool to choose the formatting, but it's likely to be culture-invariant.

### <a id="msg-links"></a>Messages with embedded links

SARIF messages can include hyperlinks to web sites as well as to constructs within the SARIF file itself.
We call these hyperlinks <a href="5.2-Glossary.md#embedded-link">_embedded links_</a>.
Both text messages and Markdown messages can contain embedded links.

#### <a id="msg-links-text-markdown"></a>Links in text and Markdown

Although a SARIF text message cannot contain formatting, it can contain hyperlinks using a small subset of
the Markdown hyperlink syntax:

```json
{
  "text": "You can learn more about XSS attacks [here](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS))."
}
```

All SARIF viewers, even those that don't support Markdown, are expected to render these link properly, for example:

> You can learn more about XSS attacks [here](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)).

On the other hand, since a SARIF viewer that chooses to render Markdown is presumed to have a full GFM parser available,
Markdown messages can use the full link syntax.
Here's an example that uses the [full reference link](https://github.github.com/gfm/#full-reference-link) syntax:

```json
{
  "text": "You can learn more about XSS attacks [here](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)).",
  "markdown": "You can learn more about XSS attacks [here][xss].\n\n[xss]: https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)"
}
```

#### <a id="msg-links-location"></a>Links to locations

SARIF supports a special "link target" syntax that refers to a location mentioned anywhere in the current result.
That's a bit abstract-sounding, so let's look at an example
(see [../samples/3-Beyond-basics/link-to-location.sarif](../samples/3-Beyond-basics/link-to-location.sarif)):

```json
{
  "ruleId": "PY2335",
  "message": {
    "text": "Use of tainted variable 'expr' (which entered the system [here](1)) in the insecure function 'eval'."
  },
  "locations": [
    {
      "physicalLocation": {
        "artifactLocation": {
          "uri": "3-Beyond-basics/bad-eval.py"
        },
        "region": {
          "startLine": 4
        }
      }
    }
  ],
  "relatedLocations": [
    {
      "id": 1,
      "message": {
        "text": "The tainted data entered the system here."
      },
      "physicalLocation": {
        "artifactLocation": {
          "uri": "3-Beyond-basics/bad-eval.py"
        },
        "region": {
          "startLine": 3
        }
      }
    }
  ]
}
```

See how the target of the embedded link in the message consists of a single integer.
This tells a SARIF viewer that the target of the link is a location defined somewhere in
this result whose `id` property matches that integer.
In this example, when a user clicks the link, the viewer should navigate to line 3
in `bad-eval.py`.<sup><a href="#note-2">2</a></sup>

Typically these links refer to elements of the `relatedLocations` array
(see ["Related locations"](#related-locations)),
but they can point to a location anywhere in the result, for example, in a
[code flow](#code-flows).

Naturally it's illegal to have more than one location in the same result with the same id.

#### <a id="msg-links-sarif-scheme"></a>Links using the `sarif:` URI scheme

## <a id="invocations"></a>Invocations

We've seen that a `run` object describes a single invocation of a single analysis tool.
We've seen that the required `run.tool` property describes the tool that produced the run.
Now we'll discuss the optional `run.invocations` property, which describes how the tool was invoked.

`run.invocations` is an _array_ of `invocation` objects.
Since a `run` describes a single invocation, you probably wonder why `run.invocations` is an array.
The spec explains:<sup><a href="#note-3">3</a></sup>

> A `run` object **MAY** contain a property named `invocations` whose value is an array of zero or more
> `invocation` objects (§3.20) that together describe a single run of a single analysis tool.
>
> **NOTE**: Normally, an analysis tool runs as a single process, and the `invocations` array requires only one element.
> The `invocations` property is defined as an array, rather than as a single `invocation` object,
> to accommodate tools which execute a sequence of programs to produce results.
> For example, a tool might run one program to determine the set of artifacts to analyze and another program
> to analyze those artifacts.
> The elements of the invocations array **SHOULD**, as far as possible, be arranged in chronological order according to
> the start time of each process. If some of the processes run in parallel, this might not be possible.

In other words, this is another case where support for an advanced scenario complicates the format.

Note that the spec clearly states that the elements of the array together describe a single run of a single tool.
It's not correct for it to contain (for example) an invocation of Clang Static Analyzer followed by
an invocation of ESLint,
or two successive invocations of Clang Static Analyzer.

The only required property of the `invocation` object is the Boolean `executionSuccessful`.
It's required because you can't tell from the integer `exitCode` property whether the tool succeeded or failed:
not every tool returns 0 on success and non-zero on failure.

There are properties to capture the command line, both as a single string (`commandLine`) and parsed into
arguments (`arguments`).
There are properties for the start and end time (`startTimeUtc` and `endTimeUtc`<sup><a href="#note-4">4</a></sup>),
for machine and environment information (`machine`, `account`, `processId`, `workingDirectory`,
`environmentVariables`),
and to capture the standard IO streams (`stdin`, `stdout`, `stderr`, `stdoutStderr`).

Most important, there are properties to capture "notifications" produced by the tool.
We'll discuss those next.

If you capture the command line, or if you use the environment-related properties like `machine`, `account`,
and `environmentVaribles`, be aware that they can contain sensitive information.
SARIF offers a facility to "redact" sensitive information, and you should become familiar with it.<sup><a href="#note-5">5</a></sup>

## <a id="notifications"></a>Notifications

In addition to analysis results, many tools provide information about their execution,
such as progress notifications (for example, "Execution started." or "Analyzing directory src/io...")
and error conditions (for example, "Rule CA1304 threw an exception and has been disabled." or
"Rule CA9999 cannot be enabled because it does not exist.").

### <a id="exec-config-notif"></a>Tool execution notification and tool configuration notifications

SARIF distinguishes two types of notifications,
<a href="5.2-Glossary.md#config-notif">_tool configuration notifications_</a> and
<a href="5.2-Glossary.md#exec-notif">_tool execution notifications_</a>.

Tool configuration notifications provide information about the configuration of the tool,
for example, which options were selected, or which rules were enabled or disabled.
They are found in the optional `invocation.toolConfigurationNotifications` property.

Tool execution notifications provide information about runtime conditions encountered
during the tool's execution,
such as the analysis start and end times, or an exception encountered during the evaluation of a rule.
They are found in the optional `invocation.toolExecutionNotifications` property.

Both `toolConfigurationNotifications` and `toolExecutionNotifications` are arrays of `notification`
objects.

Part of the rationale for distinguishing these notification types is that they are of interest to different audiences.
Tool configuration notifications are of interest to build engineers;
tool execution notifications (especially those that report runtime exceptions) are of interest to tool authors.
Of course both might be of interest to tool users, and on smaller teams the same people might fill all of these roles.

### <a id="notif-result"></a>Notifications _vs._ results

Notifications and results have much in common:

- They can both have a severity level (for example, "Analysis started."
is a `"note"`-level notification, which "Rule CA1304 threw an exception." is an `"error"`-level notification.)
- They both have user-facing messages, possibly parameterized (for example, `"Rule {0} threw an exception."`).
- Both can be described by additional metadata such as a full description, a help URI, and so forth.

For this reason, SARIF uses the same object to describe both rule metadata
and what the spec refers to as <a href="5.2-Glossary.md#notification-metadata">_notification metadata_</a>:
the `reportingDescriptor`.<sup><a href="#note-6">6</a></sup>

Note that SARIF does _not_ use the same object to represent the results and notifications themselves:
a `result` object is not the same as a `notification` object.
This is because there are so many properties of a `result` (for example, `codeFlows`) that don't apply to
notifications.

So in our [simple example](1-Introduction.md#simple-example-file),
the property `tool.driver.rules` was actually an array of `reportingDescriptor`s,
and `tool.driver` has an additional property `notifications` that is also an array of `reportingDescriptor`s.

## <a id="taxonomies"></a>Taxonomies

## <a id="code-flows"></a>Code flows

## <a id="automation"></a>Automation

## Notes

<a id="note-1"></a>1. CAUTION: `message` objects appear throughout the SARIF format, not just in `result` objects.
So it's not always appropriate to look up the message string in `tool.driver.rules`.
Depending on context, the string might come from
<a href="5.2-Glossary.md#notification-metadata">_notification metadata_</a>
(see <a href="#notifications">Notifications</a>)
or even from `globalMessageStrings`, which we won't say more about (See
[§3.19.22, globalMessageStrings property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012511)).
You can find the complete, complicated algorithm in
[§3.11.7, Message string lookup](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012424).
But you're not likely to need it, because it works as (one hopes!) as you would expect:
result messages are looked up in rule metadata,
notification messages are looked up in notification metadata,
and other messages are looked up in `globalMessageStrings`.
The algorithm is complicated because it also has to see if the message string was specified "inline" in
`message.text`,
it has to choose between the text and Markdown forms of the message,
and it has to decide whether the message was defined by the tool's driver or by one of its extensions.

<a id="note-2"></a>2. There is a potential ambiguity here that the spec doesn't explicitly address:
namely, that `1` is a perfectly fine relative URI, and you might have analyzed a file named `1`.
A SARIF producer that uses links to locations should be aware of that, and should disambiguate
file names that look like integers by (for example) changing `"1"` to `"./1"`.

<a id="note-3"></a>3. See
[§3.14.11, invocations property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012451)

<a id="note-4"></a>4. All times in SARIF's first-class properties are expressed in UTC, and the properties are named
to remind you of that.

<a id="note-5"></a>5. See
[§3.5.2, Redactable strings](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012393),
[§3.14.28, redactionTokens property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012468),
and search for the string "redactable" in the spec to find all the tokens that might contain sensitive information.

<a id="note-6"></a>6. Before Michael Fanning noticed the similarities between results and notifications,
this object was called simply the `rule` object.
This is a case where in my opinion the generalization of a concept led to a name that was less understandable.
But despite my reputation for being a good "namer," I've never been able to come up with a better one.

[Table of contents](../README.md#contents)
