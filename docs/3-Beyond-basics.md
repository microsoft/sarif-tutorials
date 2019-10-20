[Table of contents](../README.md#contents)

# Beyond the Basics

## <a id="more-about-messages"></a>More about messages

## <a id="invocations"></a>Invocations

We have seen that a `run` object describes a single invocation of a single analysis tool.
We've seen that the required `run.tool` property describes the tool that produced the run.
Now we'll discuss the optional `run.invocations` property, which describes how the tool was invoked.

`run.invocations` is an _array_ of `invocation` objects.
Since a `run` describes a single invocation, why is `run.invocations` an array?
The spec explains:<sup><a href="#note-1">1</a></sup>

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

Note that the spec clearly states that the elements of the array together describe a single run of a single tool.
It's not correct for it to contain (for example) an invocation of Clang Static Analyzer followed by
an invocation of ESLint,
or two successive invocations of Clang Static Analyzer.

The only required property of the `invocation` object is the Boolean `executionSuccessful`.
It's required because you can't tell from the integer `exitCode` property whether the tool succeeded or failed:
not every tool returns 0 on success and non-zero on failure.

There are properties to capture the command line, both as a single string (`commandLine`) and parsed into
arguments (`arguments`).
There are properties for the start and end time (`startTimeUtc` and `endTimeUtc`<sup><a href="#note-2">2</a></sup>),
for machine and environment information (`machine`, `account`, `processId`, `workingDirectory`,
`environmentVariables`),
and to capture the standard IO streams (`stdin`, `stdout`, `stderr`, `stdoutStderr`).

Most important, there are properties to capture "notifications" produced by the tool.
We'll discuss those next.

If you capture the command line, or if you use the environment-related properties like `machine`, `account`,
and `environmentVaribles`, be aware that they can contain sensitive information.
SARIF offers a facility to "redact" sensitive information, and you should become familiar with it.<sup><a href="#note-3">3</a></sup>

## <a id="notifications"></a>Notifications

## <a id="taxonomies"></a>Taxonomies

## <a id="code-flows"></a>Code flows

## Notes

<a id="note-1"></a>1. See
[§3.14.11, invocations property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012451)

<a id="note-2"></a>2. All times in SARIF's first-class properties are expressed in UTC, and the properties are named
to remind you of that.

<a id="note-3"></a>3. See
[§3.5.2, Redactable strings](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012393),
[§3.14.28, redactionTokens property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012468),
and search for the string "redactable" in the spec to find all the tokens that might contain sensitive information.

[Table of contents](../README.md#contents)
