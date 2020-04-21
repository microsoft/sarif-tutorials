[Table of contents](../README.md#contents)

# Beyond the Basics

## <a id="related-locations"></a>Related locations

Sometimes there are places in the code other than the result location that can can help you understand a problem.
We call these places <a href="Glossary.md#related-location">_related locations_</a>.
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
and might produce a `result` like this (see [bad-eval.sarif](../samples/3-Beyond-basics/bad-eval.sarif)):

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
(see [bad-eval-related-locations.sarif](../samples/3-Beyond-basics/bad-eval-related-locations.sarif)):

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
it also explains why the flagged construct is considered questionable,
provides guidance for remedying the problem,
and explains when it's ok to ignore the result.
[Appendix A](Authoring-rule-metadata-and-result-messages.md) provides guidance on authoring
informative and actionable result messages.

To avoid repeating the lengthy message in every result, a `message` object can specify an
identifier for the message text
(see [message-from-metadata.sarif](../samples/3-Beyond-basics/message-from-metadata.sarif)):

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
If rule metadata is available, a tool (or a <a href="Glossary.md#post-processor">_post-processor_</a>)
can choose to inline the messages or to refer to them in the metadata.

### <a id="msg-args"></a>Messages with arguments

Some messages vary from result to result because (for example) they mention specific variables:

```text
Variable 'x' was used without being initialized.
```

Messages in metadata can include C#-like placeholders (`{0}`).
If an analysis tool creates `message` objects that refer to message strings in metadata,
it must provide values for the placeholders by populating the `arguments` property
(see [message-with-arguments.sarif](../samples/3-Beyond-basics/message-with-arguments.sarif)):

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
We call these hyperlinks <a href="Glossary.md#embedded-link">_embedded links_</a>.
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
(see [link-to-location.sarif](../samples/3-Beyond-basics/link-to-location.sarif)):

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
This tells a SARIF viewer that the link target is a location defined somewhere in
this result whose `id` property matches that integer.
In this example, when a user clicks the link, the viewer should navigate to line 3
in `bad-eval.py`.<sup><a href="#note-2">2</a></sup>

Typically these links refer to elements of the `relatedLocations` array
(see ["Related locations"](#related-locations)),
but they can point to a location anywhere in the result, for example, in a
[code flow](#code-flows).

Naturally it's illegal to have more than one location in the same result with the same id.

## <a id="invocations"></a>Invocations

We've seen that a `run` object describes a single invocation of a single analysis tool.
We've seen that the required `run.tool` property describes the tool that produced the run.
Now we'll discuss the optional `run.invocations` property, which describes how the tool was invoked.

`run.invocations` is an _array_ of `invocation` objects.
Since a `run` describes a single invocation, you probably wonder why `run.invocations` is an array.
The spec explains:<sup><a href="#note-3">3</a></sup>

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
<a href="Glossary.md#config-notif">_tool configuration notifications_</a> and
<a href="Glossary.md#exec-notif">_tool execution notifications_</a>.

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
and what the spec refers to as <a href="Glossary.md#notification-metadata">_notification metadata_</a>:
the `reportingDescriptor`.<sup><a href="#note-6">6</a></sup>

Note that SARIF does _not_ use the same object to represent the results and notifications themselves:
a `result` object is not the same as a `notification` object.
This is because there are so many properties of a `result` (for example, `codeFlows`) that don't apply to
notifications.

So in our [simple example](1-Introduction.md#simple-example-file),
the property `tool.driver.rules` was actually an array of `reportingDescriptor`s,
and `tool.driver` has an additional property `notifications` that is also an array of `reportingDescriptor`s.

## <a id="taxonomies"></a>Taxonomies

In the context of code analysis, a <a href="Glossary.md#taxonomy">_taxonomy_</a> is a system that classifies
analysis results into a set of categories.
The SARIF spec uses the term <a href="Glossary.md#standard-taxonomy">_standard taxonomy_</a>
for a taxonomy defined independently of any particular analysis tool,
and <a href="Glossary.md#custom-taxonomy">_custom taxonomy_</a>
for a taxonomy defined by a tool.<sup><a href="#note-7">7</a></sup>
The [Common Weakness Enumeration (CWE)](https://cwe.mitre.org/) is a well-known example of a standard taxonomy.

In a sense, an analysis tool's rule set defines a taxonomy,
but the SARIF spec uses the term only for classification systems other than analysis rule sets.

SARIF can represent taxonomies and can associate results with <a href="Glossary.md#taxon">_taxa_</a>
(the individual categories within a taxonomy).

This will be easier to understand with an example. In the example below
(see [standard-taxonomy.sarif](../samples/3-Beyond-basics/standard-taxonomy.sarif)),
the analysis tool claims to support the CWE taxonomy.
The log file includes only the CWE taxa that are relevant to the results in the log file.
The tool's analysis rule `CA2101` detects memory leaks that correspond to the "Memory Leak"
taxon in the CWE taxonomy.

```json
{
  "version": "2.1.0",
  "runs": [
    {
      "taxonomies": [
        {
          "name": "CWE",
          "version": "3.2",
          "releaseDateUtc": "2019-01-03",
          "guid": "A9282C88-F1FE-4A01-8137-E8D2A037AB82",
          "informationUri": "https://cwe.mitre.org/data/published/cwe_v3.2.pdf/",
          "downloadUri": "https://cwe.mitre.org/data/xml/cwec_v3.2.xml.zip",
          "organization": "MITRE",
          "shortDescription": {
            "text": "The MITRE Common Weakness Enumeration"
          },
          "taxa": [
            {
              "id": "401",
              "guid": "10F28368-3A92-4396-A318-75B9743282F6",
              "name": "Memory Leak",
              "shortDescription": {
                "text": "Missing Release of Memory After Effective Lifetime"
              },
              "defaultConfiguration": {
                "level": "warning"
              }
            }
          ],
          "isComprehensive": false
        }
      ],
      "tool": {
        "driver": {
          "name": "CodeScanner",
          "supportedTaxonomies": [
            {
              "name": "CWE",
              "guid": "A9282C88-F1FE-4A01-8137-E8D2A037AB82"
            }
          ],
          "rules": [
            {
              "id": "CA2101",
              "shortDescription": {
                "text": "Failed to release dynamic memory."
              },
              "relationships": [
                {
                  "target": {
                    "id": "401",
                    "guid": "A9282C88-F1FE-4A01-8137-E8D2A037AB82",
                    "toolComponent": {
                      "name": "CWE",
                      "guid": "10F28368-3A92-4396-A318-75B9743282F6"
                    }
                  },
                  "kinds": [
                    "superset"
                  ]
                }
              ]
            }
          ]
        }
      },
      "results": [
        {
          "ruleId": "CA2101",
          "message": {
            "text": "Memory allocated in variable 'p' was not released."
          },
          "taxa": [
            {
              "id": "401",
              "guid": "A9282C88-F1FE-4A01-8137-E8D2A037AB82",
              "toolComponent": {
                "name": "CWE",
                "guid": "10F28368-3A92-4396-A318-75B9743282F6"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

Let's look at this fairly complicated example from top to bottom.

Each element of `run.taxonomies` describes a standard taxonomy,<sup><a href="#note-8">8</a></sup>
in this case the CWE taxonomy.
The array elements are `toolComponent` objects,
the same kind of object as `tool.driver` and the array elements of `tool.extensions`.<sup><a href="#note-9">9</a></sup>

`toolComponent.taxa` defines the individual categories defined by the taxonomy;
in this case, the single array element describes the CWE "Memory Leak" category.
The array elements are `reportingDescriptor` objects,
the same kind of object as the elements of `tool.driver.rules` and `tool.driver.notifications`.<sup><a href="#note-10">10</a></sup>

The log file does not have to include the complete taxonomy;
it only needs to include the taxa relevant to the results in the current run.
In this example, the value `false` for `toolComponent.isComprehensive` tells the SARIF consumer
that this object contains only a subset of the taxa defined by the taxonomy.<sup><a href="#note-11">11</a></sup>
(`false` is actually the default value, which makes sense because a tool should have to make an explicit statement
that it has provided the entire taxonomy.)

Moving down to the `tool` object, we see `tool.driver.supportedTaxonomies`,
which in this example says that this tool supports the CWE taxonomy.
The array elements of `supportedTaxonomies` are `toolComponentReference` objects,
which makes sense since the taxonomies themselves are `toolComponent` object.
The `toolComponentReference.guid` property matches the `guid` property in `run.taxonomies[0]`,
the object that defines the taxonomy itself.

Now let's look at `tool.driver.rules`.
Recall that each array element is a `reportingDescriptor` object, which in this context represents an analysis rule.
For the first time we encounter `reportingDescriptor.relationships`,
each of whose elements is a `reportingDescriptorRelationship` object<sup><a href="#note-12">12</a></sup>
which establishes a relationship from this rule to another `reportingDescriptor` object.
The target of the relationship can be a taxon in a taxonomy (as in this example),
or another rule within this or another tool component.

The `reportingDescriptorRelationship.target` property identifies the target of the relationship.
Its value is a `reportingDescriptorReference` object that identifies a `reportingDescriptor` within a `toolComponent`.
`reportingDescriptorReference.toolComponent`, in turn, is a `toolComponentReference` object
(which we met earlier as elements of `supportedTaxonomies`).
All together, this `reportingDescriptorReference` object designates weakness 401 in the CWE taxonomy.

Finally, `reportingDescriptorRelationship.kinds` describes the type of this relationship (there can be more than one).
In this example, the `"superset"` relationship kind tells us that CWE weakness 401 is a superset of this rule:
that is, every violation of this rule is an example of CWE weakness 401,
but not necessarily _vice versa_.<sup><a href="#note-13">13</a>, <a href="#note-14">14</a></sup>

At last we come to `run.results`, where we see that each result can have a property `taxa` specifying the categories
into which this result falls.
Like `reportingDescriptort.taxa`
(but unlike `toolComponent.taxa`, which is an array of `reportingDescriptor` objects!),
`result.taxa` is an array of `reportingDescriptorReference` objects.<sup><a href="#note-15">15</a></sup>

In this example, the result is a violation of rule `CA2101`,
and we've already seen that CWE weakness 401 is a superset of `CA2101`.
So we could infer that this result fell into the taxon "CWE 401" even if the SARIF file didn't explicitly say so.
And indeed, the spec says, in its usual formal language,
that we can omit this element of `result.taxa` in this case.<sup><a href="#note-16">16</a></sup>

## <a id="code-flows"></a>Code flows

Some tools detect issues by simulating the execution of a program,
sometimes across multiple threads of execution.
SARIF refers to the set of locations encountered in such a simulated execution as a
<a href="Glossary.md#code-flow">_code flow_</a>.
A SARIF code flow contains one or more <a href="Glossary.md#thread-flow">_thread flows_</a>
each of which describes a time-ordered sequence of code locations on a single thread of execution.<sup><a href="#note-17">17</a></sup>

Since more than one code flow might be relevant to understanding a result,
the optional `result.codeFlows` property contains an array of `codeFlow` objects.

Here's an example with a single code flow tracing a single thread of execution.
Suppose you analyze this Python file, similar to the example in <a href="#related-locations">Related locations</a>,
but with a function call for added interest
(see [bad-eval-with-code-flow.py](../samples/3-Beyond-basics/bad-eval-with-code-flow.py)):

```python
# 3-Beyond-basics/bad-eval-with-code-flow.py

print("Hello, world!")
expr = input("Expression> ")
use_input(expr)

def use_input(raw_input):
    print(eval(raw_input))
```

The tool might produce something like this (see [bad-eval-with-code-flow.sarif](../samples/3-Beyond-basics/bad-eval-with-code-flow.sarif)):

```json
{
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "PythonScanner"
        }
      },
      "results": [
        {
          "ruleId": "PY2335",
          "message": {
            "text": "Use of tainted variable 'raw_input' in the insecure function 'eval'."
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "3-Beyond-basics/bad-eval-with-code-flow.py"
                },
                "region": {
                  "startLine": 8
                }
              }
            }
          ],
          "codeFlows": [
            {
              "message": {
                "text": "Tracing the path from user input to insecure usage."
              },
              "threadFlows": [
                {
                  "locations": [
                    {
                      "message": {
                        "text": "The tainted data enters the system here."
                      },
                      "location": {
                        "physicalLocation": {
                          "artifactLocation": {
                            "uri": "3-Beyond-basics/bad-eval-with-code-flow.py"
                          },
                          "region": {
                            "startLine": 3
                          }
                        }
                      },
                      "state": {
                        "expr": {
                          "text": "undef"
                        }
                      },
                      "nestingLevel": 0
                    },
                    {
                      "location": {
                        "physicalLocation": {
                          "artifactLocation": {
                            "uri": "3-Beyond-basics/bad-eval-with-code-flow.py"
                          },
                          "region": {
                            "startLine": 4
                          }
                        }
                      },
                      "state": {
                        "expr": {
                          "text": "42"
                        }
                      },
                      "nestingLevel": 0
                    },
                    {
                      "message": {
                        "text": "The tainted data is used insecurely here."
                      },
                      "location": {
                        "physicalLocation": {
                          "artifactLocation": {
                            "uri": "3-Beyond-basics/bad-eval-with-code-flow.py"
                          },
                          "region": {
                            "startLine": 8
                          }
                        }
                      },
                      "state": {
                        "raw_input": {
                          "text": "42"
                        }
                      },
                      "nestingLevel": 1
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

We see that `result.codeFlows` is an array containing a single `codeFlow` object,
whose required `threadFlows` property in turn is an array containing a single `threadFlow` object.
The optional `codeFlow.message` explains the significance of the code flow.
(`threadFlow` also has an optional `message` property, not used in this example.)
The `threadFlow` object has a required `locations` property whose value is an array of `threadFlowLocation` objects.

Let's look more closely at the `threadFlowLocation` object.
It has an optional `message` property that describes the significance of the location.

Next, we see that it has a `location` property of type `location`,
so the code flow can include both physical and logical location
information.<sup><a href="#note-18">18</a>, <a href="#note-19">19</a></sup>

Next we have the `state` property,
whose purpose is to allow a SARIF viewer to provide a "debugger Watch window"-like experience
as the user steps through the code flow.
The property names can be anything,
but they are typically variable names (like the `"expr"` in this example)
or expressions such as `"x + y"` in the syntax of the language being analyzed.
The property values are string representations of the values of the corresponding variables or expressions.
For example, if the value of the variable `n` is the integer `42`
and the value of the variable `s` is the string `"Hello"`,
then the state property would look like this:

```json
"state": {
  "n": {
    "text": "42"
  },
  "s": {
    "text": "\"Hello\""
  }
}
```

Note that the property values aren't simple strings;
they are actually `multiformatMessageString` objects,
which we haven't met before and won't discuss further &mdash;
except to say that this allows a tool to produce a Markdown version of the expression value
for viewers that can render Markdown.

Finally we see the optional `nestingLevel` property,
whose purpose is to allow a SARIF viewer to present an indented view of the execution trace, for example:

```text
bad-eval-with-code-flow:3
bad-eval-with-code-flow:4
    bad-eval-with-code-flow:8
```

The level of indentation would typically track the function call nesting level,
but the spec doesn't require that.<sup><a href="#note-20">20</a></sup>

We've just scratched the surface of code flows in SARIF.
The `codeFlow`, `threadFlow`, and `threadFlowLocation` objects all have more properties for more advanced scenarios.

## <a id="automation"></a>Automation

You can run a SARIF-producing analysis tool by hand at any time,
examine the resulting log file in a text editor or in a SARIF viewer,
and use the results to improve your code.
But SARIF goes beyond "manual" usage scenarioes with features that support its usage in large teams with elaborate,
automated engineering processes.

SARIF provides ways to uniquely identify a run and to describe its role in the user's
<a href="Glossary.md#engineering-system">_engineering system_</a>.
Note that it is the _run_, not the _log file_, that has a unique identity.
The log file is just a packaging mechanism for a set of runs;
it's the runs that are important.<sup><a href="#note-21">21</a></sup>

### <a id="run-ids"></a>Run identifiers

A run is identified by the `run.automationDetails` property,
whose name suggests its intended usage: to enable automatic processing of scan results in an engineering system.
Its value is an `runAutomationDetails` object.
Here's an example (see [automation-details.sarif](../samples/3-Beyond-basics/automation-details.sarif)):

```json
{
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "CodeScanner"
        }
      },
      "automationDetails": {
        "description": {
          "text": "This is the October 10, 2018 nightly run of the CodeScanner tool on all product binaries in the 'master' branch of the 'sarif-sdk' repo"
        },
        "id": "CodeScanner/nightly/sarif-sdk/master/2018-10-05",
        "guid": "d541006e-582d-4600-a603-64925b7f7f35",
        "correlationGuid": "53819b2e-a790-4f8b-b68f-a145c13b4f39"
      },
      "results": []
    }
  ]
}
```

#### <a id="run-id-and-guid"></a>The `id` and `guid` properties

Depending on the needs of your engineering system, you can identify your runs with a string-valued `id` property,
a `guid` property, or both.

The `id` property is what the spec refers to as a
<a href="Glossary.md#hierarchical-string">_hierarchical string_</a>.<sup><a href="#note-22">22</a></sup>
In a hierarchical string, the slashes are significant,
and they're interpreted as defining a logical hierarchy.
This means (for example) that a viewer is allowed to display a list of run ids indented by that hierarchy,
or that an API is allowed to support a query such as "find all runs under `CodeScanner/nightly/sarif-sdk`".

The spec explicitly states when a string-valued property is hierarchical.
In string-valued properties that are not hierarchical (which is most of them),
slashes are not special and SARIF consumers aren't allowed to assume that they define
a logical hierarchy.<sup><a href="#note-23">23</a></sup>

#### <a id="run-correlationGuid"></a>The `correlationGuid` property

## Notes

<a id="note-1"></a>1. CAUTION: `message` objects appear throughout the SARIF format, not just in `result` objects.
So it's not always appropriate to look up the message string in `tool.driver.rules`.
Depending on context, the string might come from
<a href="Glossary.md#notification-metadata">_notification metadata_</a>
(see <a href="#notifications">Notifications</a>)
or even from `globalMessageStrings`, which we won't say more about (see
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

<a id="note-7"></a>7. See [§3.19.3, Taxonomies](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012492).

<a id="note-8"></a>8. If a tool supported a custom taxonomy,
that taxonomy would appear as `tool.driver.taxa`
(or `tool.extensions[].taxa` if the custom taxonomy were defined by a tool extension).
This is a much less common scenario than the use of a standard taxonomy such as CWE,
and we won't discuss it further.

<a id="note-9"></a>9. The reason for this design choice was that almost all of the properties of this object make sense
both for the tool's driver and extensions and for taxonomies.
The result of this choice is that the spec occasionally has to include text
to describe the conditions where certain properties can or cannot appear
(see, for example, [§3.19.25, taxa property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012514)):

> If the `toolComponent` describes a standard taxonomy (for example, the Common Weakness Enumeration [CWE™]),
it **SHALL NOT** contain `rules` (§3.19.23) or `notifications` (§3.19.24).

<a id="note-10"></a>10. Again, the reason for this choice was the almost complete overlap between the properties that
make sense for rules, notifications, and taxa.
Again, the result is that occasionally the spec has to describe differences among the usages of the same object
for different purposes
(see, for example, [§3.49.11,
messageStrings property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012803)):

> If the `reportingDescriptor` object defines a rule,
the set of property names appearing in the `messageStrings` property **SHALL** contain at least the set of strings
which occur as values of `result.message.id` properties (§3.27.11, §3.11.10) in the current run object...
>
> If the `reportingDescriptor` object describes a notification,
the set of property names appearing in the `messageStrings` property **SHALL** contain at least the set of strings
which occur as values of `notification.message.id` for any notification object in the run.

<a id="note-11"></a>11. To avoid repeating taxonomy definitions in every log file,
and to provide access to the complete taxonomy without bloating the log file,
SARIF provides a facility called <a href="Glossary.md#external-property-file">_external property files_</a>
(see [§3.15.2](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012471))
that allows large data sets needed by a SARIF log file to be stored in separate files.

<a id="note-12"></a>12. See [§3.49.11,
reportingDescriptorRelationship object](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012826).

<a id="note-13"></a>13. For information about all kinds of reporting descriptor relationships, see
[§3.53.3,
kinds property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012829).

<a id="note-14"></a>14. The "directionality" of the relationships is confusing.
Reading the log file, it appears to say that "Rule `CA2101` is a superset of CWE weakness 401,"
when in fact it's the other way around.
The TC made this choice because the relationship kind definitions came from an existing tool,
and the TC felt that it would confuse users of that tool if SARIF's definitions were the opposite of the tool's.

<a id="note-15"></a>15. Admittedly this is a bit confusing to remember.
The SARIF TC considered using distinct names,
so that the properties in `reportingDescriptor` and `result` would have been named (for example) "`taxonReferences`"
instead of "`taxa`".
We eventually decided that the more concise name was preferable.
Actually the TC went through a little naming exercise at the end where we changed several properties names
to improve conciseness.
This might surprise you if you read the spec and encounter a name like `externalPropertyFileReference`!

<a id="note-16"></a>16. As
[§3.27.8,
taxa property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012602)
so eloquently puts it:

> `thisObject.taxa` does not need to contain elements which correspond to `superset` or `equals` relationships;
rather, the result **SHALL** implicitly be taken to fall into all the taxa described by those relationships.

<a id="note-17"></a>17. The spec does not commit to any particular operating system implementation of the concept of
"thread of execution"
([§3.36.1, Code flow object, General](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012697)):

> We define a thread flow as a temporally ordered sequence of code locations occurring within a single thread of execution,
> typically an operating system thread or a fiber.

<a id="note-18"></a>18. Here's another example of where reasonable people might disagree about the property naming.
The array elements of `threadFlow.locations` are of type `threadFlowLocation` (_not_ of type `location`),
whereas the property `threadFlowLocation.location` _is_ of type `location`.
Perhaps `threadFlow.locations` should have been named `threadFlow.threadFlowLocations`,
but again, the TC chose conciseness in naming.

<a id="note-19"></a>19. `threadFlowLocation.location` is almost always present, but it's not required because
there are rare circumstances where location information is not available.
See [§3.38.3, location property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012710)
for such an example.

<a id="note-20"></a>20. Here's what the spec actually says about `threadFlowLocation.nestingLevel` (see
[§3.38.10, nestingLevel property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012717)):

> [It] represents any type of logical containment hierarchy among the `threadFlowLocation` objects
> in the `threadFlow`. Typically, it represents function call depth.
>
> A viewer that renders a `threadFlow` **SHOULD** provide a visual representation of the value of `nestingLevel`.
> Typically, this would be an indentation indicating the depth of each location in the call tree.

<a id="note-21"></a>21. The SARIF spec doesn't say these words.
This is simply a point of view I advocated in the TC design meetings,
whenever we argued over whether a particular property should appear on the `run` object or the `sarifLog` object.
The fact that the `sarifLog` object has hardly any properties of its own beyond the `runs` array indicates
that this point of view prevailed.

<a id="note-22"></a>22. See
[§3.5.4,
Hierarchical strings](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012395)

<a id="note-23"></a>23. Within a team's engineering system,
there might be out of band information (such as a convention)
that interprets other string-valued properties as hierarchical strings,
and tools built to work within that engineering system might respect that hierarchy.
But the log files won't be interoperable, in the sense that "generic" SARIF tools &mdash;
tools _not_ specifically built to work within that engineering system &mdash;
won't recognize the hierarchy.

[Table of contents](../README.md#contents)
