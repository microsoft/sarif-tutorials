[Table of contents](../README.md#contents)

# Introduction

## <a id="what-is-sarif"></a>What is SARIF?

SARIF, the Static Analysis Results Interchange Format, is a standard, JSON-based format for the output of
<a href="Glossary.md#static-analysis-tool">_static analysis tools_</a>.
It is an approved [OASIS](https://www.oasis-open.org/)
[Committee Specification](https://www.oasis-open.org/news/announcements/static-analysis-results-interchange-format-sarif-v2-1-0-from-the-sarif-tc-is-an-a),<sup><a href="#note-1">1</a></sup>
and is on the way to becoming a full OASIS specification.

SARIF is a rich format intended to meet the needs of sophisticated tools,
while still being practical for use by simpler tools.
Because it would be impractical to support every feature of every tool,
SARIF provides an extensibility mechanism to allow tool authors to store custom data that the SARIF format
doesn't directly represent.

## <a id="tools"></a>About static analysis tools

Static analysis tools look for <a href="Glossary.md#issue">_issues_</a>
by examining a program without executing it.<sup><a href="#note-2">2</a></sup>
We often refer to a static analysis tool simply as a "tool."

We can classify tools according to the language they analyze and the kinds of issues they detect.
For example:

- A security analyzer detects errors that might cause security vulnerabilities.
- An accessibility checker detects user interface constructs that might not be accessible to people with
limited sight or motor impairments.
- A geopolitical checker detects terms or images that might be politically or culturally sensitive
in certain countries or cultures.
- A linter detects uses of error-prone or dangerous programming language constructs.
- A style checker detects violations of a team's coding guidelines.
- A binary analyzer might detect the use of an outdated compiler or insecure compilation options.

Because there are so many kinds of issues, and so many programming languages in common use,
there are thousands of static analysis tools,
most of them targeted at a particular class of issues in a particular programming language.

## <a id="why-sarif"></a>Why SARIF?

Historically, every static analysis tool has defined its own output format.
These formats are frequently based on standard file formats such as XML or JSON,
but beyond that, they have little in common &mdash; or at least, not enough to make it feasible
for automated systems to consume all the different formats that exist.

This matters because engineering teams, especially large ones, can use dozens of tools.
The multiplicity of output formats leads to many problems:

- Users have to learn to read each tool's output format.
- There is no common way to view and interact with the tool outputs in the user's development environment
(for example, in Visual Studio, VS Code, or Eclipse).
- There is no common way to convert the tool outputs into bugs in an issue-tracking system such as GitHub or
Azure Dev Ops.
- There is no common way to generate metrics (for example, how many bugs exist in each program component).

That's where SARIF comes in.
By providing a common tool output format, SARIF reduces the learning burden on users,
and makes it possible to create common tooling for all tools:
viewers, bug filers, metrics calculators, _etc._.

## <a id="simple-example"></a>A simple example

Here's a simple example to give you the flavor of SARIF.

ESLint is a linter for ECMAScript (a.k.a. JavaScript) that accepts "formatter" plugins so it can produce
output in a variety of formats.
The [ESLint SARIF formatter](https://www.npmjs.com/package/eslint.formatter.sarif) plugin produces the SARIF format.

Consider this simple JavaScript file:

```javascript
var x = 42
```

If we run ESLint with the SARIF formatter:

```shell
.\node_modules\.bin\eslint --format sarif simple-example.js --output-file simple-example.sarif
```

... we get:

<a id="simple-example-file"></a>

```json
{
  "version": "2.1.0",
  "$schema": "http://json.schemastore.org/sarif-2.1.0-rtm.4",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "ESLint",
          "informationUri": "https://eslint.org",
          "rules": [
            {
              "id": "no-unused-vars",
              "shortDescription": {
                "text": "disallow unused variables"
              },
              "helpUri": "https://eslint.org/docs/rules/no-unused-vars",
              "properties": {
                "category": "Variables"
              }
            }
          ]
        }
      },
      "artifacts": [
        {
          "location": {
            "uri": "file:///C:/dev/sarif/sarif-tutorials/samples/Introduction/simple-example.js"
          }
        }
      ],
      "results": [
        {
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
          ],
          "ruleId": "no-unused-vars",
          "ruleIndex": 0
        }
      ]
    }
  ]
}
```

Without knowing any more about SARIF, we can see that:

- The log file was produced by the tool ESLint.
- The tool scanned the file `simple-example.js`.
- The scan produced a single result: a violation of the rule `no-unused-vars` at line 1, column 5.

Looking more closely, we can see that SARIF can provide additional information about the tool
(the `informationUri` property) and about the analysis rules it defines (the `rules` array,
including a `helpUri` property for each rule).

There are also many things we don't understand yet:

- What do the properties `index` and `ruleIndex` mean?
- What is an "artifact"? Isn't it just a file? Why not call it that?
- Why are there so many levels of nesting? For example:
  - Why do we have `tool.driver` instead of just `tool`?
  - Why do we have `message.text` instead of just `message`?
  - Why do we have `location.physicalLocation` instead of just `location`?
  - For that matter, why is `locations` an array?

We'll learn later that the answer to most of these questions is "to support more advanced scenarios".

## <a id="plan"></a>The plan for these tutorials

Now that we have a little background, let's dive into the format.
We'll explore the basic concepts first, then move on to more advanced concepts.

As I've said, SARIF is a large, powerful format that addresses the needs of sophisticated tools of many kinds.

But if you're a tool author, your tool probably needs to produce only a small subset of the information that SARIF can represent.
(In fact, many of the properties in the simple example above are optional, but I wanted to show you the output of
an actual tool.)
The goal of this tutorial is to help you quickly to understand the features of SARIF that everyone needs,
and to find the more advanced features that make sense for your tool.

Even if you're writing a SARIF viewer (say, an Eclipse plugin),
which ideally would be able to visualize everything in a SARIF file,
you don't have to do it all at once.
You can start with a viewer that just displays the basics,
and move on to displaying the more advanced features as your customers require.
The trick is to know what the basics _are_.
That's where these tutorials come in.

In short, you don't need to know everything about SARIF.
Just read the introductory material, then pick and choose the additional topics that matter to you
and the tools you need to write, or the log files you need to read.

Good luck, and I hope SARIF makes your experience with static analysis tools better and more productive!

## Notes

<a id="note-1">1.</a> This means that it has been approved by the OASIS SARIF Technical Committee,
but not yet by OASIS as a whole.

<a id="note-2">2.</a> A tool that looks for issues in a program by observing the program's execution
is called a <a href="Glossary.md#dynamic-analysis-tool">_dynamic analysis tool_</a>.
SARIF does not claim to represent the output of dynamic analysis tools,
but there are dynamic analysis tools that have adopted it successfully. YMMV.

[Table of contents](../README.md#contents)
